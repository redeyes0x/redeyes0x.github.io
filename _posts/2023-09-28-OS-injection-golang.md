---
layout: "post"
title: "OS Command injections in exec.LookPath golang"
date: 2023-03-5T21:44:02+01:00
---

> <html><body><b><p style="color:#ffffff;font-size:25px">Introduction:</p></b></body></html>

<div dir="rtl" align="right">
السلام عليكم ورحمة الله وبركاته
</div>

<div dir="rtl" align="right">
لغة golang اللغة المحببة عند غالبية مختبرين الاختراق ومن اسرع واكثر اللغات البرمجية المستخدمة حاليًا في برامج الطرفية او برامج cli, لكن كإي لغة برمجه لاتخلوا من الثغرات لأن ليس هنالك شيء امن بنسبة 100% وفي مثالنا هذا سنشرع في شرح إحدى الثغرات التي وجدت في مكتبة os/exec
</div>

> <html><body><b><p style="color:#ffffff;font-size:25px">OS Command injections in exec.LookPath():</p></b></body></html>

<div dir="rtl" align="right">
الثغرة اللي راح نتطرق لها في موضوعنا من نوع (OS command injections).
</div>

<div dir="rtl" align="right">
غالبًا ثغرات الحقن او injection تكون ناتجه عن ثقة المبرمج بمدخلات المستخدم ولكن, مثالنا اليوم مختلف قليلاً في حالتنا الداله المصابة هي exec.LookPath() وهي من مكتبة os/exec في لغة golang, الدالة هذي بكل اختصار تتحق من وجودالباينري في $PATH, وهذا يسهل على المطور اذا هي موجوده يخزنها في متغير وفي حالتنا راح يكون `PATH` وبكذا يقدر يستدعيه من اي مكان واذا حصل خطأ ينبه مستخدم البرنامج برسالة الخطأ لكن كمختبرين اختراق راح نستغل هذا الشيء وبكذا اظن وضحة لكم الفكرة.
</div>

<div dir="rtl" align="right">
</div>

```golang
package main //هذا الكود مثال للاصابة سنشرح عليه

import (
	"fmt"
	"os/exec"
)

func main() {
	path, err := exec.LookPath("ls") //<--- موضع الاصابة
	if err != nil {
		fmt.Printf("Failed to find 'ls' command: %v\n", err)
		return
	}
	fmt.Printf("The 'ls' command is located at: %s\n", path)
}
```

<div dir="rtl" align="right">
لو نلاحظ السطر التاسع راح يستدعي امر `ls` من الباث اذا كان موجود راح يطبع The 'ls' command is located at واذا ماكان الباينري موجود راح يطبع Failed to find 'ls' command.
كشخص عادي تشوف الموضوع بسيط لكن كمختبر اختراق وفاهم لناظم لينكس تعرف انك تستطيع اضافة اي مجلد في PATH environment انا راح اضيف مثلاً sshpass واقول للبرنامج ابحث عن sshpass td $PATH فلو تشوفون الصورة المرفقة
</div>

<img src="/img/1-sshpass-in-path.png" alt="centered image" />


<div dir="rtl" align="right">
فهو مالقى sshpass في المسارات الطبيعية في الصورة اللي فوق, اذا كتبنا امر  echo $PATH راح تظهر لنا جميع المسارات الافتراضية الموجوده
</div>

<img src="/img/2-PATH.png" alt="centered image" />


<div dir="rtl" align="right">
ممتاز لكن غالبًا اذا المبرمج استخدم دالة exec.LookPath() بالعادة راح يستخدم معه دالة تنفذ الاوامر وهذا منطقي وشفته في عدت مشاريع  وهذا الكود التالي بعد ماضبطته ليحاكي الواقع وايضاً بعدها النتيجة
</div>

<img src="/img/3-code.png" alt="centered image" />

<img src="/img/4-id.png" alt="centered image" />

> <html><body><b><p style="color:#ffffff;font-size:25px">Where the issue?:</p></b></body></html>

<div dir="rtl" align="right">
المشكلة بكل بساطه لو فرضنا البرنامج المستخدم برنامج مساعد مثل sshpass وكان هذا البرنامج غير موجود في جهاز المستخدم 

يستطيع المهاجم استغلال هذا الخطأ لصالحه بإنشاء  ملف بسم sshpass ويضع فيه أكواد  الاستغلال ومن ثم اضافة الملف مثلاً  في مكان مثل tmp وبعد هذا يضع مسار ال tmp في $PATH لو تشوفون الصوره التاليه علشان تتضح الصوره </div>

<img src="/img/4-passwd.png" alt="centered image" />

<div dir="rtl" align="right">
طيب ليش تعتبر هذي ثغرة ؟ بكل بساطه لأن الداله تبحث عن  الباينري كمتغير وليس كمسار كامل للبرنامج

 اذا المطور استخدم مثلاً برنامج /usr/bin/nmap هنا لو شخص انشئ باينري بإسم nmap ووضعه في tmp وسوا نفس السيناريو الي فوق

ماراح تعتبر ثغرة لأن المبرمج حدد مسار البرنامح بتحديد لكن في مثالنا يعتبر ثغرة والسبب أستغلال عدم تأكد دالة exec.LookPath() من مسار البرنامج

وليش قلت كذا السبب حتى مطوريين لغة  go أعترفو بهذي الثغره وبحط روابط مهمه تشرح خطورة الثغره 
</div>

> <html><body><b><p style="color:#ffffff;font-size:25px">Vulnerable software:</p></b></body></html>


<div dir="rtl" align="right">
البرامج المصابه بهذي الثغرة كثير لكن بذكر ثنتين وحده لها CVE والثانيه لقيتها في برنامج راح ارفق لكم الروابط:
</div>

-  https://github.com/xcodeOn1/Vault-exploit

-  https://nvd.nist.gov/vuln/detail/CVE-2020-26284

<div dir="rtl" align="right">
ملحوظة : 
برنامج  vault بلغتهم بالثغره لكن ماعتبروها ثغره فا ماعلي ولا عليكم حرج أعتبروها لاب Xd 

طبعاُ بعد عندكم hugo وتلقى في القت هب مليان اذا حاب تجرب الثغره 

</div>

> <html><body><b><p style="color:#ffffff;font-size:25px">CTF Lab:</p></b></body></html>

<div dir="rtl" align="right">
في تحدي في echoCTF مبني على سيناريو هذي الثغرة انصحكم فيه وهذا رابطه:
</div>

-  https://echoctf.red/target/167


> <html><body><b><p style="color:#ffffff;font-size:25px">References:</p></b></body></html>

- https://github.com/golang/go/issues/38736

- https://go.dev/blog/path-security


**Twitter** :

```
https://twitter.com/xcode0x
```