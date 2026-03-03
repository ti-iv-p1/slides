---
title: Emmet (Visual Studio Code)
version: 1.0.0
header: Emmet (Visual Studio Code)
footer: https://github.com/ti-iv-p1/slides/emmet
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Emmet (Visual Studio Code)

---

Emmet adalah fitur di Visual Studio Code (VS Code) yang memungkinkan penulisan kode HTML dan CSS lebih cepat dengan menggunakan ekspansi singkatan (abbreviations).

---

## **1. Struktur Dasar HTML (`!` atau `html:5`)**

**Singkatan:**

```html
!
```

atau

```html
html:5
```

**Ekspansi:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    ....
  </body>
</html>
```

**Fungsi:**  
Membuat template dasar HTML dalam satu ketikan.

---

## **2. Membuat Elemen HTML**

### **a) Elemen Dasar**

**Singkatan:**

```html
div
```

**Ekspansi:**

```html
<div></div>
```

**Fungsi:**  
Membuat elemen `<div>` dengan cepat.

---

### **b) Elemen dengan Class (`.`)**

**Singkatan:**

```html
div.container
```

**Ekspansi:**

```html
<div class="container"></div>
```

**Fungsi:**  
Membuat elemen `<div>` dengan class `container`.

---

### **c) Elemen dengan ID (`#`)**

**Singkatan:**

```html
section#hero
```

**Ekspansi:**

```html
<section id="hero"></section>
```

**Fungsi:**  
Membuat elemen `<section>` dengan `id="hero"`.

---

### **d) Elemen Bersarang (`>`)**

**Singkatan:**

```html
nav>ul>li
```

**Ekspansi:**

```html
<nav>
  <ul>
    <li></li>
  </ul>
</nav>
```

**Fungsi:**  
Membuat elemen yang bersarang (parent-child relationship).

---

## **3. Duplikasi Elemen (`*`)**

**Singkatan:**

```html
ul>li*5
```

**Ekspansi:**

```html
<ul>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
</ul>
```

**Fungsi:**  
Membuat **5 buah** elemen `<li>` di dalam `<ul>`.

---

## **4. Menambahkan Teks Placeholder (`{}`)**

**Singkatan:**

```html
p{Ini adalah paragraf contoh}
```

**Ekspansi:**

```html
<p>Ini adalah paragraf contoh</p>
```

**Fungsi:**  
Menambahkan teks langsung ke dalam elemen.

---

## **5. Membuat Beberapa Elemen dengan Class dan ID**

**Singkatan:**

```html
div#main.container
```

**Ekspansi:**

```html
<div id="main" class="container"></div>
```

**Fungsi:**  
Membuat elemen `<div>` dengan **id="main"** dan **class="container"**.

---

## **6. Menggunakan Grup (`()`)**

**Singkatan:**

```html
div>(header>h1)+section+footer
```

**Ekspansi:**

```html
<div>
  <header>
    <h1></h1>
  </header>
  <section></section>
  <footer></footer>
</div>
```

**Fungsi:**  
Membuat struktur yang kompleks dengan grup elemen.

---

## **7. Elemen Bersaudara (`+`)**

**Singkatan:**

```html
h1+p
```

**Ekspansi:**

```html
<h1></h1>
<p></p>
```

**Fungsi:**  
Membuat elemen **bersaudara** (sibling elements).

---

## **8. Menambahkan Atribut (`[]`)**

**Singkatan:**

```html
input[type="text" placeholder="Masukkan nama"]
```

**Ekspansi:**

```html
<input type="text" placeholder="Masukkan nama" />
```

**Fungsi:**  
Menambahkan atribut langsung ke elemen.

---

## **9. Elemen Auto-numbering (`$`)**

**Singkatan:**

```html
ul>li.item$*3
```

**Ekspansi:**

```html
<ul>
  <li class="item1"></li>
  <li class="item2"></li>
  <li class="item3"></li>
</ul>
```

**Fungsi:**  
Menambahkan angka otomatis pada class.
