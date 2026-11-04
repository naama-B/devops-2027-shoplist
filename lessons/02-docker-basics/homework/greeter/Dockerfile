# בסיס: image של פייתון בגרסה מדויקת, לא latest
FROM python:3.12-slim
# מעתיקים קודם רק את קובץ התלויות, כדי שה-cache ישמר כשהקוד משתנה
COPY requirements.txt .
# התקנת התלויות (flask)
RUN pip install -r requirements.txt
# מעתיקים את הקוד אחרי התלויות, כי הוא משתנה הכי הרבה
COPY app.py .
# תיעוד: האפליקציה מאזינה בפורט 8000 (לא פותח כלום בפועל)
EXPOSE 8000
# הרצת האפליקציה בצורת exec, עם סוגריים מרובעים
CMD ["python", "app.py"]
