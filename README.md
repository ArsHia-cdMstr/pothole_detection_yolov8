# Road Pothole Detection using YOLOv8 (Custom Segmentation)

## مقدمه

این پروژه با هدف تشخیص و سگمنت‌سازی چاله‌های جاده‌ای (potholes) طراحی شده است.  
با استفاده از مدل YOLOv8 می‌توان چاله‌ها را در تصاویر و ویدیوها شناسایی و برای هر چاله ماسک پیکسلی یا bounding box تولید کرد. خروجی‌های عددی مانند مساحت تقریبی هر چاله نیز قابل محاسبه است که برای اولویت‌بندی تعمیرات و مدیریت نگهداری راه‌ها مفید است.

## Google Colab Notebook

برای اجرای تمام مراحل دانلود دیتاست، آموزش و اینفرنس با یک کلیک و روی GPU، نوت‌بوک زیر را در Colab باز کنید:

[`Google Colab File`](https://colab.research.google.com/drive/17SLXw-wdHG2syQhLSHH5r5_rkZx5poo0)

## خروجی‌ها (Features)

- تشخیص چاله‌ها با bounding box و/یا ماسک سگمنتیشن.
- خروجی اینفرنس به صورت تصویر یا ویدیو با نمایش نتایج.
- وزن آموزش‌دیده (best.pt) برای استقرار یا استفاده مجدد.
- امکان محاسبه مساحت تقریبی چاله‌ها از روی ماسک‌های باینری.

## تکنولوژی‌ها

- Python 3
- ultralytics (YOLOv8)
- OpenCV
- Roboflow (دیتاست)
- Jupyter Notebook / Google Colab

## پیش‌نیازها

- کارت گرافیک با درایور مناسب برای آموزش سریع (در Colab می‌توانید از GPU استفاده کنید).
- Python 3.8+ و pip

## نصب و راه‌اندازی (Quick Start)

1. کلون کردن مخزن:

```bash
git clone https://github.com/ArsHia-cdMstr/pothole_detection_yolov8.git
cd pothole_detection_yolov8

```

2. نصب وابستگی‌ها:

```bash
pip install ultralytics
pip install roboflow
pip install opencv-python

```

3. دانلود دیتاست (نمونه با Roboflow):

```python
from roboflow import Roboflow
rf = Roboflow(api_key="YOUR_API_KEY")
project = rf.workspace("YOUR_WORKSPACE").project("pothole-detection-project")
dataset = project.version(1).download("yolov8")

```

توجه: اگر از Colab استفاده می‌کنید، در نوت‌بوک Runtime را روی GPU قرار دهید و در صورت تمایل درایو گوگل را mount کنید.
`

## آموزش مدل (Training)

مثال اجرای آموزش با مدل از پیش‌آموزش‌دیده yolov8m:
bash
yolo task=detect mode=train model=yolov8m.pt data=dataset/data.yaml epochs=50 imgsz=640
پارامترها (قابل تنظیم):

- model: yolov8n/s/m/l بر اساس منابع و دقت مورد نظر
- epochs: تعداد epochها (برای دیتاست کوچک مقدار کم‌تر یا fine-tune توصیه می‌شود)
- imgsz: اندازه ورودی (مثلاً 640)

## پیش‌بینی (Inference)

اجرای inference روی ویدیو یا تصویر:
bash
yolo task=detect mode=predict model=runs/detect/train/weights/best.pt conf=0.25 source='demo.mp4'
خروجی‌ها در مسیر runs/detect/predict یا مشابه آن ذخیره می‌شوند.

## محاسبه مساحت تقریبی از ماسک

در صورتی که ماسک باینری برای هر چاله در اختیار باشد، می‌توان تعداد پیکسل‌های ناحیه را شمرد و با مقیاس صحیح به مساحت تبدیل کرد. مثال ساده:
python
import cv2

mask = cv2.imread("mask.png", cv2.IMREAD_GRAYSCALE)
area_pixels = cv2.countNonZero(mask)

# pixel_area_m2 باید بر اساس مقیاس تصویر محاسبه شود (مثال: هر پیکسل معادل 0.0001 مترمربع)

pixel_area_m2 = 0.0001
area_m2 = area_pixels * pixel_area_m2
print("Estimated pothole area (m^2):", area_m2)
توضیح: تبدیل پیکسل به واحد واقعی نیازمند دانستن ارتفاع دوربین، فواصل و رزولوشن واقعی تصویر یا استفاده از نقاط مرجع است.

## نکات ارزیابی و محدودیت‌ها

- کیفیت و کمیت دیتاست تأثیر بسیار زیادی بر عملکرد مدل دارد. دیتاست کوچک یا نامتوازن منجر به mAP و دقت پایین خواهد شد.
- معیارهای مرسوم: Precision, Recall, mAP50, mAP50-95 و F1-score.
- برای دیتاست‌های کوچک:
   - از data augmentation استفاده کنید (Mosaic, flip, color jitter و غیره).
   - ابتدا فقط head را fine-tune کنید یا از learning rate کوچک استفاده کنید.

- برچسب‌گذاری با کیفیت و یکنواخت ضروری است (خطا در annotations عملکرد را کاهش می‌دهد).

## پیشنهادات برای بهبود پروژه

- جمع‌آوری و برچسب‌زنی تصاویر بیشتر با شرایط نوری و زاویه‌های متنوع.
- مقایسه بین نسخه‌های مختلف YOLOv8 (n, s, m, l) و انتخاب مدل متناسب با محدودیت سخت‌افزاری.
- افزودن ماژول گزارش‌گیری اتوماتیک (خروجی CSV یا داشبورد) با اطلاعات مساحت، مختصات و زمان/مکان (در صورت داشتن GPS).
- توسعه یک رابط ساده وب (Flask یا Streamlit) برای آپلود عکس و مشاهده نتایج توسط کاربران غیر فنی.
- تبدیل مدل برای استقرار روی دستگاه‌های لبه (تبدیل به ONNX یا TensorRT).

## منابع

- Ultralytics YOLOv8: https://github.com/ultralytics/ultralytics
- Roboflow : https://app.roboflow.com/

---

تهیه شده توسط: ارشیا مخلص


