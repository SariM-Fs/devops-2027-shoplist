# שיעור 6 - תשובות

## 1
לא השתמשנו ב-docker-backend כי זה שם ש-Compose נתן לבד, לא אנחנו בחרנו אותו, ולא נוח לעבוד איתו. עדיף שם ברור שאנחנו בוחרים, עם תג.
בלי imagePullPolicy: Never, אם Kubernetes לא מוצא את ה-image במחשב, הוא מנסה להוריד אותו מהאינטרנט. ה-image שלנו לא קיים שם, אז ה-pod נכשל עם ErrImagePull.

## 2
הרצנו פענוח של base64:
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("c2hvcHBhc3M="))
ויצא shoppass.
base64 זה לא הצפנה, זו רק הצורה שבה Kubernetes רוצה שיכתבו, וכל אחד יכול לפענח. לכן Secret לא באמת מאובטח בעצמו: כל מי שרואה את הקובץ יודע את הסיסמה, ובעבודה אמיתית לא שמים קובץ כזה עם סיסמה אמיתית ב-Git.

## 3
הקובץ שאחראי לזה הוא postgres-pvc.yml. ה-PVC הוא כמו ארון ששומר את כל המידע, והוא מחובר ל-postgres בתיקייה /var/lib/postgresql/data. גם אם העובד נופל ולוקח איתו את השולחן, זה לא משנה, כי המידע נמצא בארון והעובד החדש מקבל את אותו ארון.
הפקודה שכן הייתה מוחקת את המוצרים: kubectl delete pvc postgres-pvc -n shoplist, כי היא מוחקת את הארון עצמו.

## 4
אם ה-backend עולה לפני postgres, הוא לא מצליח להתחבר למסד הנתונים. ה-readinessProbe בודק כל כמה שניות את /health, וכל עוד הבדיקה נכשלת ה-pod הוא 0/1 וה-Service לא שולח אליו בקשות. כש-postgres מוכן, הבדיקה עוברת, ה-pod הופך ל-1/1 ומתחיל לקבל בקשות.

## 5
בדפדפן הנורית של backend הייתה אדומה והמוצרים לא נטענו. ה-pods של backend לא נפלו, הם נשארו 1/1 Running. get endpoints הראה <none>, כי ה-selector של ה-Service חיפש את התג app: backnd, ועל ה-pods כתוב app: backend. התגים לא תאמו, אז ה-Service לא מצא אף pod להעביר אליו בקשות.

## 6
משתני סביבה נקראים רק כשה-pod עולה. שינוי ב-ConfigMap לא משנה pods שכבר רצים, והם ממשיכים עם הערך הישן. הפתרון הוא להפעיל מחדש את ה-pods, כדי שהחדשים יקראו את הערך החדש:
kubectl rollout restart deployment/backend -n shoplist

## 7
בכל הקבצים כתבנו shoplist, כי בכל קובץ צריך לכתוב namespace: shoplist כדי שכולם יהיו באותו namespace (וגם ב-namespace.yml וב-pg_isready). כדי להריץ עותק שני ב-shoplist-test נצטרך לשנות את כל הקבצים, וזה לא נוח וקל לטעות ולשכוח אחד. עדיף שהשם יהיה כתוב במקום אחד בלבד, וכל הקבצים יקחו ממנו.