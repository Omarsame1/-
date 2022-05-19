
# سيعمل سير العمل هذا على إنشاء تطبيق node.js ودفعه إلى تطبيق Azure Web عندما يتم دفع الالتزام إلى الفرع الافتراضي الخاص بك.
#
# يفترض سير العمل هذا أنك قمت بالفعل بإنشاء تطبيق ويب Azure App Service المستهدف.
# للحصول على إرشادات ، راجع https://docs.microsoft.com/en-us/azure/app-service/quickstart-nodejs؟tabs=linux&pivots=development-environment-cli
#
# لتكوين سير العمل هذا:
#
# 1. قم بتنزيل ملف تعريف النشر لتطبيق Azure Web. يمكنك تنزيل هذا الملف من صفحة "نظرة عامة" الخاصة بتطبيق الويب الخاص بك في Azure Portal.
#     لمزيد من المعلومات: https://docs.microsoft.com/en-us/azure/app-service/deploy-github-actions؟
#
# 2. أنشئ سرًا في المستودع الخاص بك باسم AZURE_WEBAPP_PUBLISH_PROFILE ، الصق محتويات ملف تعريف النشر كقيمة للسر.
#     للحصول على إرشادات حول الحصول على ملف تعريف النشر ، راجع: https://docs.microsoft.com/azure/app-service/deploy-github-actions#configure-the-github-secret
#
# 3. قم بتغيير قيمة AZURE_WEBAPP_NAME. اختياريًا ، قم بتغيير متغيرات البيئة AZURE_WEBAPP_PACKAGE_PATH و NODE_VERSION أدناه.
#
# لمزيد من المعلومات حول إجراءات GitHub لـ Azure: https://github.com/Azure/Actions
# لمزيد من المعلومات حول إجراء نشر تطبيقات الويب Azure: https://github.com/Azure/webapps-deploy
# لمزيد من العينات لبدء استخدام مهام سير عمل GitHub Action للنشر في Azure: https://github.com/Azure/actions-workflow-samples

في :
  دفع :
    الفروع :
      - رئيسي
  workflow_dispatch :

البيئة المحيطة :
  AZURE_WEBAPP_NAME : your-app-name     # اضبط هذا على اسم التطبيق الخاص بك
  AZURE_WEBAPP_PACKAGE_PATH : ' .       # عيّن هذا على المسار إلى مشروع تطبيق الويب الخاص بك ، وافتراضيًا إلى جذر المستودع
  NODE_VERSION : '14 .x '                 # اضبط هذا على إصدار العقدة المراد استخدامه

أذونات :
  المحتويات : قراءة

الوظائف :
  بناء :
    يعمل على : ubuntu-latest
    خطوات :
    - الاستخدامات : الإجراءات / الخروج @ v3

    - الاسم : إعداد Node.js
      يستخدم : الإجراءات / عقدة الإعداد @ v3
      مع :
        إصدار العقدة : $ {{env.NODE_VERSION}}
        ذاكرة التخزين المؤقت : " npm "

    - الاسم : npm install and build and test
      تشغيل : |
        تثبيت npm
        npm تشغيل البناء - إذا كان موجودًا
        اختبار تشغيل npm - إذا كان موجودًا
    - الاسم : تحميل الأداة لمهمة النشر
      الاستخدامات : الإجراءات / تحميل الأداة @ v3
      مع :
        الاسم : تطبيق العقدة
        المسار :. _

  نشر :
    أذونات :
      المحتويات : لا يوجد
    يعمل على : ubuntu-latest
    الاحتياجات : البناء
    البيئة :
      الاسم : " تطوير "
      url : $ {{steps.deploy-to-webapp.outputs.webapp-url}}

    خطوات :
    - الاسم : تنزيل الأداة من بناء الوظيفة
      الاستخدامات : الإجراءات / download-artifact @ v3
      مع :
        الاسم : تطبيق العقدة

    - الاسم : " النشر إلى Azure WebApp "
      المعرّف : النشر إلى تطبيق الويب
      الاستخدامات : azure / webapps -loy @ v2
      مع :
        اسم التطبيق : $ {{env.AZURE_WEBAPP_NAME}}
        نشر الملف الشخصي : $ {{secrets.AZURE_WEBAPP_PUBLISH_PROFILE}}
        الحزمة : $ {{env.AZURE_WEBAPP_PACKAGE_PATH}}
