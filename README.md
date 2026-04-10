DRF: `models.py` fayli va `ModelSerializer` metodlarini mashq qilishingiz uchun masalalar.
Serializer kodini o'zingiz yozasiz.

---

## 1. `models.py` – namunaviy modellar

```python
from django.db import models


class Author(models.Model):
    full_name = models.CharField(max_length=200)
    birth_date = models.DateField()
    email = models.EmailField(unique=True)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.full_name


class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)
    description = models.TextField(blank=True)

    def __str__(self):
        return self.name


class Book(models.Model):
    title = models.CharField(max_length=250)
    author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name='books')
    categories = models.ManyToManyField(Category, related_name='books')
    price = models.DecimalField(max_digits=10, decimal_places=2)
    published_date = models.DateField(null=True, blank=True)
    is_bestseller = models.BooleanField(default=False)
    stock_count = models.PositiveIntegerField(default=0)
    
    # Slug maydoni (avtomatik yoki qo'lda to'ldiriladi)
    slug = models.SlugField(max_length=250, unique=True, blank=True)

    def __str__(self):
        return self.title
```

---

## 2. `ModelSerializer` metodlari uchun masalalar

Quyidagi masalalarni yechishda `serializers.py` faylida kerakli serializerlarni yozing. Har bir masala alohida serializer sinfini talab qilishi mumkin.

### Masala 1 – Asosiy ModelSerializer
**Vazifa:** `Book` modeli uchun `BookSerializer` yarating. Barcha maydonlar (`title`, `author`, `categories`, `price`, `published_date`, `is_bestseller`, `stock_count`, `slug`) serializerda bo'lsin. Faqat `slug` maydoni **faqat o'qish uchun** (`read_only`) bo'lsin.

### Masala 2 – `create()` metodini qayta yozish
**Vazifa:** `AuthorSerializer` yarating. Unda `full_name`, `birth_date`, `email`, `is_active` maydonlari bo'lsin. `create()` metodini shunday qayta yozingki, agar `email` unikal bo'lmasa, u holda avvalgi muallifning `is_active` qiymatini `False` ga o'zgartirib, yangi muallifni yaratsin.

### Masala 3 – `update()` metodini qayta yozish
**Vazifa:** `BookSerializer` da `update()` metodini qayta yozing. Agar kitobning narxi (`price`) o'zgartirilsa va yangi narx eskisidan 50% dan ko'proq farq qilsa, o'zgartirishni **taqiqlang** (ValidationError chiqaring). Agar farq 50% dan kam yoki teng bo'lsa, yangilashga ruxsat bering.

### Masala 4 – Field‑level validation
**Vazifa:** `BookSerializer` da `validate_price` metodini yozing. Narx 0 dan kichik bo'lmasligi va 10 000 dan katta bo'lmasligi kerak. Agar shart buzilsa, `ValidationError` qaytaring.

### Masala 5 – Object‑level validation
**Vazifa:** `BookSerializer` da `validate()` metodini yozing. Kitobning `published_date` kelajakdagi sana bo'lmasligi va `stock_count` 0 bo'lsa, `is_bestseller` `True` bo'la olmasligini tekshiring.

### Masala 6 – `SerializerMethodField` ishlatish
**Vazifa:** `AuthorSerializer` ga quyidagi ikkita qo'shimcha maydon qo'shing:
- `book_count` – muallifning nechta kitobi borligini ko'rsatsin.
- `latest_book_title` – muallifning eng so'nggi nashr etilgan kitobining nomini qaytarsin (`published_date` bo'yicha).  
Buning uchun `SerializerMethodField` dan foydalaning.

### Masala 7 – Nested serializer va `create()` murakkab holat
**Vazifa:** `Book` yaratishda `author` ma'lumotlarini **ichma-ich** (nested) jo'natish mumkin bo'lsin.  
`BookCreateSerializer` yarating:  
- `author` maydoni `AuthorSerializer` (kichik) shaklida bo'lsin.  
- `categories` maydoni IDlar ro'yxati (`PrimaryKeyRelatedField`) sifatida qabul qilinsin.  
- `create()` metodini yozing: agar berilgan muallif (email bo'yicha) mavjud bo'lmasa, yangi muallif yaratsin; mavjud bo'lsa, shu muallifni ishlatsin. So'ng kitob va unga bog'liq kategoriyalarni yaratsin.

### Masala 8 – Dinamik maydonlarni boshqarish (`__init__` orqali)
**Vazifa:** `BookDynamicSerializer` yozing. Konstruktor (`__init__`) orqali `fields` parametrini qabul qilsin. Agar `fields` berilgan bo'lsa, faqat shu maydonlarni ko'rsatsin; aks holda barcha maydonlarni chiqarsin. Masalan:  
`BookDynamicSerializer(instance=book, fields=['title', 'price'])`

### Masala 9 – `to_representation()` ni qayta yozish
**Vazifa:** `BookSerializer` da `to_representation()` metodini o'zgartiring. Chiqishda `author` maydoni o'rniga muallifning `full_name` qiymatini `author_name` kaliti bilan qaytaring. `categories` o'rniga kategoriya nomlari ro'yxatini `category_names` kaliti bilan qaytaring.

### Masala 10 – ModelSerializerda `validators` va `unique_together` ga o'xshash tekshiruv
**Vazifa:** `Book` modelida (`author`, `title`) juftligi unikal bo'lishi kerak. Buni serializer darajasida `validators` yoki `validate()` yordamida amalga oshiring. `UniqueTogetherValidator` dan foydalanishingiz mumkin.

---
