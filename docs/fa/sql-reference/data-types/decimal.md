---
machine_translated: true
machine_translated_rev: d734a8e46ddd7465886ba4133bff743c55190626
toc_priority: 42
toc_title: "\u062F\u0647\u062F\u0647\u06CC"
---

# Decimal(P, S) Decimal32(ع) Decimal64(ع) Decimal128(S) {#decimalp-s-decimal32s-decimal64s-decimal128s}

امضا اعداد ثابت نقطه که دقت در طول اضافه کردن نگه دارید, تفریق و ضرب عملیات. برای تقسیم رقم حداقل قابل توجهی دور انداخته می شوند (گرد نیست).

## پارامترها {#parameters}

-   ص دقیق. محدوده معتبر: \[ 1 : 38 \]. تعیین می کند که چگونه بسیاری از اعداد اعشاری می تواند داشته باشد (از جمله کسر).
-   ص-مقیاس. محدوده معتبر: \[0: پ \]. تعیین می کند که چگونه بسیاری از رقم اعشاری کسر می تواند داشته باشد.

بسته به فسفر اعشاری مقدار پارامتر (پ, بازدید کنندگان) مترادف برای است:
- پ از \[ 1 : 9 \] - برای اعشار (بازدید کنندگان)
- پ از \[ 10 : 18 \] - برای اعشار64(بازدید کنندگان)
- پ از \[ 19: 38 \] - برای اعشار128 (بازدید کنندگان)

## محدوده مقدار دهدهی {#decimal-value-ranges}

-   دسیمال32 (بازدید کنندگان) - ( -1 \* 10^(9 - بازدید کنندگان), 1 \* 10^(9 بازدید کنندگان) )
-   اعشار64 (بازدید کنندگان) - ( -1 \* 10^(18 - س), 1 \* 10^(18 بازدید کنندگان) )
-   اعشار128 (بازدید کنندگان) - ( -1 \* 10^(38 - بازدید کنندگان), 1 \* 10^(38 بازدید کنندگان) )

برای مثال decimal32(4) می تواند شامل اعداد از -99999.9999 به 99999.9999 با 0.0001 گام.

## نمایندگی داخلی {#internal-representation}

داخلی داده ها به عنوان اعداد صحیح امضا نرمال با عرض بیت مربوطه نشان داده است. محدوده ارزش واقعی است که می تواند در حافظه ذخیره می شود کمی بزرگتر از بالا مشخص, که تنها در تبدیل از یک رشته بررسی.

چرا که پردازنده مدرن اعداد صحیح 128 بیتی بومی را پشتیبانی نمی کند, عملیات بر روی اعشار128 شبیه سازی. از آنجا که این decimal128 با این نسخهها کار به طور قابل توجهی کندتر از decimal32/decimal64.

## عملیات و نوع نتیجه {#operations-and-result-type}

عملیات دودویی در نتیجه اعشاری در نوع نتیجه گسترده تر (با هر سفارش از استدلال).

-   `Decimal64(S1) <op> Decimal32(S2) -> Decimal64(S)`
-   `Decimal128(S1) <op> Decimal32(S2) -> Decimal128(S)`
-   `Decimal128(S1) <op> Decimal64(S2) -> Decimal128(S)`

قوانین برای مقیاس:

-   اضافه کردن به, تفریق کردن: بازدید کنندگان = حداکثر(بازدید کنندگان 1, بازدید کنندگان2).
-   multuply: S = S1 + S2.
-   تقسیم: S = S1.

برای عملیات مشابه بین دهدهی و اعداد صحیح, نتیجه اعشاری از همان اندازه به عنوان یک استدلال است.

عملیات بین دهدهی و float32/float64 تعریف نشده. اگر شما به آنها نیاز دارید, شما می توانید به صراحت بازیگران یکی از استدلال با استفاده از todecimal32, todecimal64, todecimal128 یا tofloat32, tofloat64 برنامهنویسی. به خاطر داشته باشید که نتیجه دقت از دست دادن و تبدیل نوع یک عملیات محاسباتی گران است.

برخی از توابع در نتیجه بازگشت اعشاری به عنوان شناور64 (مثلا, ور یا استودف). محاسبات متوسط هنوز هم ممکن است در دهدهی انجام شود, که ممکن است به نتایج مختلف بین نت64 و ورودی اعشاری با ارزش های مشابه منجر شود.

## بررسی سرریز {#overflow-checks}

در طول محاسبات در اعشاری, عدد صحیح سرریز ممکن است رخ دهد. رقم بیش از حد در کسری دور انداخته می شوند (گرد نیست). رقم بیش از حد در بخش عدد صحیح به یک استثنا منجر شود.

``` sql
SELECT toDecimal32(2, 4) AS x, x / 3
```

``` text
┌──────x─┬─divide(toDecimal32(2, 4), 3)─┐
│ 2.0000 │                       0.6666 │
└────────┴──────────────────────────────┘
```

``` sql
SELECT toDecimal32(4.2, 8) AS x, x * x
```

``` text
DB::Exception: Scale is out of bounds.
```

``` sql
SELECT toDecimal32(4.2, 8) AS x, 6 * x
```

``` text
DB::Exception: Decimal math overflow.
```

سرریز چک منجر به کاهش سرعت عملیات. اگر مشخص است که سرریزهای امکان پذیر نیست, حس می کند برای غیر فعال کردن چک با استفاده از `decimal_check_overflow` تنظیمات. هنگامی که چک غیر فعال هستند و سرریز اتفاق می افتد, نتیجه نادرست خواهد بود:

``` sql
SET decimal_check_overflow = 0;
SELECT toDecimal32(4.2, 8) AS x, 6 * x
```

``` text
┌──────────x─┬─multiply(6, toDecimal32(4.2, 8))─┐
│ 4.20000000 │                     -17.74967296 │
└────────────┴──────────────────────────────────┘
```

چک سرریز اتفاق می افتد نه تنها در عملیات ریاضی بلکه در مقایسه ارزش:

``` sql
SELECT toDecimal32(1, 8) < 100
```

``` text
DB::Exception: Can't compare.
```

[مقاله اصلی](https://clickhouse.tech/docs/en/data_types/decimal/) <!--hide-->