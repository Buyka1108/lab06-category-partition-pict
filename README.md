# Лаборатори 6 — Category-Partition ба PICT

- Оюутны нэр: Ж.Буянхишиг
- Оюутны код: B232270114
- Хичээл: F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест

## Зорилго

Банкны шилжүүлгийн функцэд Category-Partition аргаар тестийн сонголт,
хязгаарлалт болон хүлээгдэх үр дүнг тодорхойлно.
Microsoft PICT ашиглан pairwise тестүүд үүсгэж, гараар боловсруулсан
тестүүдтэй харьцуулна.

## Ажлын орчин

- Ubuntu — WSL 2
- Microsoft PICT 3.7.4 — эх кодоос бүтээсэн
- PICT бүтээсэн нотолгоо: results/pict-build.txt

## A1. Сонголт ба төлөөлөх утгууд

Тестийн суурь нөхцөл:
- A100 — илгээх данс, A200 — хүлээн авах данс; хоёулаа MNT валюттай.
- Өдрийн нийт лимит: 100000 MNT.
- Fresh: өмнөх зарцуулалт 0, үлдэх лимит 100000 MNT.
- Partial: өмнөх зарцуулалт 80000, үлдэх лимит 20000 MNT.
- Тестийн тогтмол ханш: 1 USD = 3500 MNT. Энэ нь бодит ханш биш.
- Тест бүрийн өмнө данс, үлдэгдэл, өмнөх зарцуулалтыг шинээр тохируулна.

| Параметр | Сонголт | Төлөөлөх утга / нөхцөл | Тэмдэглэгээ ба үндэслэл |
|---|---|---|---|
| From | Valid | A100, идэвхтэй | [property: from_active] Хэвийн шилжүүлгийн эх данс. |
| From | Frozen | A100, царцаагдсан | [ERROR] ERROR_FROZEN алдааг шалгана. |
| From | Missing | X999, бүртгэлгүй | [ERROR] ERROR_NO_FROM алдааг шалгана. |
| To | Valid | A200, бүртгэлтэй, A100-аас өөр | [property: to_exists] Хэвийн хүлээн авах данс. |
| To | Missing | Y999, бүртгэлгүй | [ERROR] ERROR_NO_TO алдааг шалгана. |
| To | SameAsFrom | A100 → A100 | [ERROR, IF: from_exists] Бодит данс өөртөө шилжүүлэхэд ERROR_SAME_ACCOUNT. |
| Amount | Normal | 10000 MNT эсвэл 2 USD = 7000 MNT; balance=200000 | [IF: active accounts] Үлдэгдэл, үлдэх лимитийн доторх эерэг дүн. |
| Amount | Zero | 0 MNT эсвэл 0 USD | [ERROR] amount > 0 шаардлагыг зөрчинө. |
| Amount | Negative | -1 MNT эсвэл -1 USD | [ERROR] Сөрөг дүн ERROR_BAD_AMOUNT үүсгэнэ. |
| Amount | OverBalance | 10000 MNT эсвэл 2 USD; balance=5000 | [ERROR] Хөрвүүлсний дараах дүн үлдэгдлээс их боловч үлдэх лимитийн дотор. |
| Amount | OverLimit | Fresh: 110000 MNT эсвэл 32 USD; Partial: 30000 MNT эсвэл 9 USD; balance=200000 | [ERROR] Үлдэх лимитээс их боловч үлдэгдлийн дотор. |
| Amount | AtLimit | 100000 MNT; balance=200000; Fresh | [SINGLE] Яг лимиттэй тэнцүү заагийг нэг суурь нөхцөлөөр шалгана. |
| Currency | Same | MNT | [property: no_conversion] Дансны валюттай ижил тул хөрвүүлэхгүй. |
| Currency | Different | USD | [property: conversion] Дүнг MNT рүү хөрвүүлсний дараа үлдэгдэл, лимитийг шалгана. |
| LimitState | Fresh | Өмнөх зарцуулалт 0 MNT | [property: remaining=100000] Өдрийн лимит бүтэн байна. |
| LimitState | Partial | Өмнөх зарцуулалт 80000 MNT | [property: remaining=20000] Өмнөх шилжүүлгийн нөлөөг шалгана. |

Amount нь хүсэлтийн Currency валютаар өгөгдөнө.
Үлдэгдэл болон лимитийг хөрвүүлсний дараах MNT дүнгээр шалгана.
OverBalance болон OverLimit-ийн setup-ийг нэг нэгнийхээ алдааг
давхар үүсгэхгүйгээр сонгосон.

## A2. Хязгаарлалт ба боломжит хослолын тоо

### Алдааны давуу эрэмбэ

Олон алдаа зэрэг байвал дараах эрэмбийг ашиглана:

ERROR_NO_FROM → ERROR_FROZEN → ERROR_NO_TO →
ERROR_SAME_ACCOUNT → ERROR_BAD_AMOUNT →
ERROR_INSUFFICIENT → ERROR_LIMIT.

Эхлээд дансны хүчинтэй байдлыг, дараа нь дүн болон санхүүгийн
боломжийг шалгах нь ойлгомжтой тул энэ эрэмбийг сонгов.
Энэ нь бидний тест дизайнд сонгосон эрэмбэ бөгөөд анхны
тодорхойлолтод баталгаажсан хэрэгжүүлэлтийн зан төлөв биш.

### Хязгаарлалтууд

| № | Хязгаарлалт | Үндэслэл |
|---|---|---|
| 1 | From=Missing бол To=Valid | Эх данс байхгүй алдааг бусад дансны алдаанаас тусгаарлана. |
| 2 | From≠Valid бол Amount=Normal, Currency=Same, LimitState=Fresh | Эх дансны алдааг дүн, валют, лимитийн нөхцөлөөс тусгаарлана. |
| 3 | From=Frozen бол To=Valid эсвэл Missing | Frozen + Missing гэсэн зориудын давхар алдааны тестийг хадгална. |
| 4 | To≠Valid бол Amount=Normal, Currency=Same, LimitState=Fresh | Хүлээн авах дансны алдааг санхүүгийн алдаанаас тусгаарлана. |
| 5 | To=SameAsFrom бол From=Valid | Ижил дансны алдааг шалгахад хүчинтэй эх данс шаардлагатай. |
| 6 | Amount=AtLimit бол From=Valid, To=Valid, Currency=Same, LimitState=Fresh | [SINGLE] заагийн тестийг нэг суурь нөхцөлд шалгана. |

[ERROR] сонголтуудыг ихэвчлэн нэг алдаатай тестээр тусгаарлана.
Frozen + Missing нь олон алдааны эрэмбийг шалгах зориудын онцгой тест.
[SINGLE] нь AtLimit сонголтыг ганц төлөөлөх нөхцөлөөр шалгах стратеги.
PICT-д SINGLE гэсэн түлхүүр үг байхгүй тул үүнийг IF дүрмээр хэрэгжүүлнэ.

### Бүрэн олонлог

From: 3 сонголт.
To: 3 сонголт.
Amount: 6 сонголт.
Currency: 2 сонголт.
LimitState: 2 сонголт.

N = 3 × 3 × 6 × 2 × 2 = 216.

### Хязгаарлалтын дараах олонлог

| Нөхцөл | Тооцоо | Хослол |
|---|---|---:|
| From=Valid, To=Valid, Amount≠AtLimit | 5 amount × 2 currency × 2 LimitState | 20 |
| AtLimit-ийн SINGLE нөхцөл | Valid/Valid/AtLimit/Same/Fresh | 1 |
| From=Missing | Missing/Valid/Normal/Same/Fresh | 1 |
| From=Frozen | To нь Valid эсвэл Missing; бусад утга тогтмол | 2 |
| From=Valid, To нь Missing эсвэл SameAsFrom | Бусад утга Normal/Same/Fresh | 2 |
| Нийт | 20 + 1 + 1 + 2 + 2 | 26 |

Иймээс N → M: **216 → 26**.

26 нь бүх хязгаарлалтыг хангасан боломжит тест frame-ийн тоо.
Энэ нь PICT-ийн pairwise аргаар гаргах тестийн мөрийн тоо биш.
Хязгаарлалтууд олон алдааны бүх хослолыг хадгалахгүй;
нэг зориудын давхар алдааны нөхцөлийг үлдээсэн.

## A3. Тестийн спецификаци

Тест бүрийн setup нь A1 хүснэгтийн төлөөлөх утгуудыг ашиглана.
Амжилтын хариу: HTTP 201, {"result":"OK"}.
Оролтын алдааны хариу: HTTP 200, {"result":"ERROR_..."}.

| ID | From | To | Amount | Currency | LimitState | HTTP | Хүлээгдэх result |
|---|---|---|---|---|---|---:|---|
| A01 | Valid | Valid | Normal | Same | Fresh | 201 | OK |
| A02 | Valid | Valid | Normal | Different | Partial | 201 | OK |
| A03 | Missing | Valid | Normal | Same | Fresh | 200 | ERROR_NO_FROM |
| A04 | Frozen | Valid | Normal | Same | Fresh | 200 | ERROR_FROZEN |
| A05 | Valid | Missing | Normal | Same | Fresh | 200 | ERROR_NO_TO |
| A06 | Valid | SameAsFrom | Normal | Same | Fresh | 200 | ERROR_SAME_ACCOUNT |
| A07 | Valid | Valid | Zero | Same | Fresh | 200 | ERROR_BAD_AMOUNT |
| A08 | Valid | Valid | Negative | Different | Partial | 200 | ERROR_BAD_AMOUNT |
| A09 | Valid | Valid | OverBalance | Different | Fresh | 200 | ERROR_INSUFFICIENT |
| A10 | Valid | Valid | OverLimit | Same | Partial | 200 | ERROR_LIMIT |
| A11 | Valid | Valid | AtLimit | Same | Fresh | 201 | OK |
| A12 | Frozen | Missing | Normal | Same | Fresh | 200 | ERROR_FROZEN |
| A13 | Valid | Valid | OverLimit | Different | Fresh | 200 | ERROR_LIMIT |

A11 нь [SINGLE] тест: дүн өдрийн лимиттэй тэнцүү тул зөвшөөрөгдөнө.
A12 нь давхар алдааны тест: эх данс царцаагдсан, авах данс байхгүй.
A2-т сонгосон эрэмбээр ERROR_FROZEN нь ERROR_NO_TO-оос түрүүлнэ.
Эдгээр нь тест дизайны хүлээгдэх үр дүн; сервер ажиллуулж шалгаагүй.
