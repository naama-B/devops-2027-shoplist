## ד8 — multi-stage

| Image | Size |
|---|---|
| shoplist-frontend:kitchen | 244MB |
| shoplist-frontend:multi | 92.7MB |
| shoplist-frontend:single | 92.8MB |


1.Compose יוצר רשת פנימית שבה כל service מוכר לפי השם שלו (DNS פנימי של Docker). stats פונה ישירות ל-`http://backend:5000` בתוך הרשת, ולכן לא צריך ports. הדפדפן לא מגיע ל-backend ישירות: הוא פונה ל-`localhost:8080`, שהוא ה-frontend (nginx), ו-nginx מעביר בקשות `/api/` אל `backend:5000` בתוך הרשת. רק frontend חשוף החוצה.
2.אחרי `down` רגיל המוצרים נשארים: הקונטיינרים והרשת נמחקים, אבל `postgres-data` הוא named volume שמנוהל על ידי Docker ושורד מחיקה של קונטיינרים. עם `down -v` גם ה-named volumes נמחקים, ולכן כל המוצרים נעלמים. ב-`up` הבא מסד הנתונים נוצר מאפס ו-`init.sql` רץ שוב ויוצר טבלה ריקה.
3.זה bind mount. יודעים לפי הצד השמאלי: הוא נתיב לקובץ במחשב (מתחיל ב-`../`), ולא שם של volume. אם משנים את `init.sql` אחרי שהמסד נוצר, לא יקרה כלום, כי Postgres מריץ את הקבצים ב-`docker-entrypoint-initdb.d` רק באתחול הראשון, כשתיקיית הנתונים ריקה. כדי שהשינוי ירוץ צריך `down -v` ואז `up`.
4.הרשת: `docker_default`. ה-volume: `docker_postgres-data`. Compose מוסיף לכל שם קידומת של שם הפרויקט, ושם הפרויקט הוא כברירת מחדל שם התיקייה שבה נמצא `docker-compose.yml`, כלומר `docker`. `default` הוא שם הרשת שנוצרת אוטומטית, ו-`postgres-data` הוא השם שהגדרנו תחת `volumes:`.
5.יש: nginx, קבצי האתר המעובדים מ-`/app/dist` (בתוך `/usr/share/nginx/html`) ו-`nginx.conf`. אין: Node, `build.js` והקבצים המקוריים, כי הם נשארו בשלב ה-build שנזרק. אין `CMD` כי ה-image `nginx` כבר מגדיר `CMD` שמפעיל את nginx, ו-image שבנוי עליו יורש אותו. ל-stats לא צריך multi-stage כי אין בו שלב בנייה: Node נדרש בזמן ריצה כדי להריץ את `server.js`, כך שאין כלים שאפשר לזרוק.
6.כתוב למעלה( טבלה)
7.nginx בוחר את ה-location עם התחילית הארוכה (הספציפית) ביותר שמתאימה. `/api/stats` ארוך יותר מ-`/api/`, ולכן הוא מנצח והבקשה עוברת ל-`stats:3000`. כשב-`proxy_pass` יש לוכסן בסוף, nginx מחליף את החלק שהתאים (`/api/`) ב-`/`, ולכן `/api/products` מגיע ל-backend בתור `/products`. בלי הלוכסן, ה-backend היה מקבל `/api/products`, נתיב שלא קיים אצלו, ומחזיר 404.
