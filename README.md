
import pandas as pd

# قراءة بيانات الذهب
df = pd.read_csv("XAUUSD_15m.csv")

# تنظيف أسماء الأعمدة
df.columns = df.columns.str.strip().str.lower()

# التأكد من وجود عمود الوقت
if "datetime" not in df.columns:
    raise ValueError(
        f"عمود datetime غير موجود. الأعمدة المتاحة: {list(df.columns)}"
    )

# تحويل الوقت وترتيب الشموع
df["datetime"] = pd.to_datetime(df["datetime"], errors="coerce")
df = df.dropna(subset=["datetime"])
df = df.drop_duplicates(subset=["datetime"])
df = df.sort_values("datetime")
df = df.set_index("datetime")

# التحقق من أعمدة الأسعار
required = ["open", "high", "low", "close"]
missing = [col for col in required if col not in df.columns]

if missing:
    raise ValueError(f"أعمدة الأسعار الناقصة: {missing}")

# تحويل الأسعار إلى أرقام
for col in required:
    df[col] = pd.to_numeric(df[col], errors="coerce")

df = df.dropna(subset=required)

# فحص سلامة الشموع
invalid = (
    (df["high"] < df["low"]) |
    (df["high"] < df["open"]) |
    (df["high"] < df["close"]) |
    (df["low"] > df["open"]) |
    (df["low"] > df["close"])
)

print("عدد الشموع:", len(df))
print("من:", df.index.min())
print("إلى:", df.index.max())
print("\nالإحصاءات:")
print(df[required].describe())
print("\nالشموع غير السليمة:", int(invalid.sum()))
print("\nآخر 5 شموع:")
print(df.tail())

