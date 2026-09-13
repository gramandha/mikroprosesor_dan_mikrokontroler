# Penjelasan Arsitektur CPU 8-bit

Gambar yang dilampirkan merupakan **diagram blok arsitektur internal CPU 8-bit**, dengan **Data Bus 8-bit** dan **Address Bus 16-bit**. Arsitektur ini sangat mirip dengan keluarga **Z80**, karena terdapat register `IX`, `IY`, `PC`, `SP`, `A`, `F`, `B`, `C`, `D`, `E`, dan `H`, `L`, serta sistem interrupt.

Secara sederhana:

> **Register → Data Bus → ALU → Register → Address Bus → Memory/I/O**

---

## 1. Data Bus — 8 bit

Bagian paling atas bertuliskan **Data Bus** dengan lebar **8 bit**.

Data bus digunakan untuk membawa **data 8-bit** antarbagian CPU.

Contoh alur:

```text
Register B
    ↓
Data Bus 8-bit
    ↓
   ALU
    ↓
Data Bus
    ↓
Register A
```

Karena lebarnya 8 bit, data yang dapat direpresentasikan adalah:

```text
00000000 sampai 11111111
```

atau:

```text
0 sampai 255
```

---

## 2. Address Bus — 16 bit

Bagian paling bawah adalah **Address Bus 16 bit**.

Address bus digunakan untuk menentukan **alamat memory** yang akan diakses CPU.

Dengan 16 bit, jumlah alamat yang dapat direpresentasikan:

\[
2^{16} = 65.536
\]

atau **64 KB alamat memory**.

Contoh alamat:

```text
0000H
0001H
0002H
...
FFFFH
```

Perbedaan utama:

| Bus | Lebar | Fungsi |
|---|---:|---|
| Data Bus | 8 bit | Membawa data |
| Address Bus | 16 bit | Menentukan alamat |

---

## 3. ALU — Arithmetic Logic Unit

Bagian berbentuk trapesium berwarna merah muda adalah **ALU**.

ALU merupakan bagian CPU yang melakukan operasi aritmetika dan logika.

### Operasi aritmetika

```text
5 + 3 = 8
10 - 4 = 6
```

### Operasi logika

```text
AND
OR
XOR
NOT
```

### Perbandingan

Misalnya:

```text
A = 10
B = 10
```

Ketika dilakukan perbandingan, ALU dapat melakukan operasi internal seperti:

\[
A-B = 10-10 = 0
\]

Hasil tersebut kemudian memengaruhi **Flags**.

---

## 4. Register A — Accumulator

Kotak `A` merupakan **Accumulator**.

Accumulator adalah register utama yang sangat sering digunakan oleh ALU.

Contoh:

```asm
LD A, 5
ADD A, 3
```

Secara sederhana:

```text
A = 5
 ↓
ALU + 3
 ↓
A = 8
```

Sehingga:

\[
A \leftarrow A+3
\]

Register A memiliki ukuran **8 bit**.

---

## 5. Register F — Flags

Kotak `F` merupakan **Flag Register**.

Register ini menyimpan status hasil operasi ALU.

Pada arsitektur seperti Z80, flag yang penting antara lain:

| Flag | Nama | Fungsi |
|---|---|---|
| S | Sign | Menunjukkan hasil bertanda negatif |
| Z | Zero | Menunjukkan hasil = 0 |
| H | Half Carry | Carry dari bit 3 ke bit 4 |
| P/V | Parity/Overflow | Paritas atau overflow |
| N | Add/Subtract | Menunjukkan operasi pengurangan |
| C | Carry | Menunjukkan adanya carry |

### Contoh Carry

Misalnya:

```text
A = 255
A + 1
```

Secara matematis:

\[
255+1=256
\]

Tetapi register 8-bit hanya dapat menyimpan 0–255.

Hasil 8-bit menjadi:

```text
00000000
```

dan:

```text
Carry Flag = 1
```

---

## 6. Register B, C, D, E, H, L

Pada sisi kanan terdapat enam register 8-bit:

```text
B   C
D   E
H   L
```

Register-register tersebut juga dapat dipasangkan menjadi register 16-bit.

### BC

```text
B = 8 bit
C = 8 bit

BC = 16 bit
```

### DE

```text
D = 8 bit
E = 8 bit

DE = 16 bit
```

### HL

```text
H = 8 bit
L = 8 bit

HL = 16 bit
```

Contoh:

```text
B = 12H
C = 34H
```

maka:

```text
BC = 1234H
```

Jadi register dapat digunakan sebagai register **8-bit** maupun pasangan **16-bit**, tergantung instruksi.

---

## 7. PC — Program Counter

**PC (Program Counter)** adalah register 16-bit.

Fungsinya menunjukkan:

> **Alamat instruksi berikutnya yang akan diambil dari memory.**

Misalnya:

```text
PC = 1000H
```

CPU akan mengambil instruksi dari:

```text
Memory[1000H]
```

Setelah instruksi diambil, PC biasanya bertambah sesuai panjang instruksi.

Contoh instruksi 1 byte:

```text
PC = 1000H
      ↓
Fetch instruction
      ↓
PC = 1001H
```

Jika instruksi memiliki panjang 2 byte:

```text
PC = 1000H
      ↓
ambil 2 byte
      ↓
PC = 1002H
```

---

## 8. SP — Stack Pointer

**SP (Stack Pointer)** adalah register 16-bit yang menunjukkan posisi **stack** di memory.

Stack digunakan untuk:

- menyimpan alamat return
- menyimpan data sementara
- menyimpan register
- menangani interrupt
- mekanisme `CALL` dan `RET`

Contoh:

```asm
CALL FUNCTION
```

CPU perlu menyimpan alamat untuk kembali setelah function selesai.

Alamat tersebut disimpan ke stack.

Kemudian:

```asm
RET
```

mengambil kembali alamat tersebut.

---

## 9. IX dan IY

`IX` dan `IY` adalah register **16-bit** yang digunakan sebagai **Index Register**.

Keduanya dapat digunakan untuk mengakses data berdasarkan alamat dasar + offset.

Misalnya:

```text
IX = 2000H
```

Kemudian CPU mengakses:

```text
(IX + 5)
```

Maka:

\[
2000H+5=2005H
\]

CPU mengakses:

```text
Memory[2005H]
```

Mekanisme ini berguna untuk mengakses:

- array
- tabel
- struktur data
- data relatif terhadap suatu alamat

---

## 10. Adder `+` untuk IX/IY

Pada diagram terdapat blok `+` di bawah IX/IY.

Blok tersebut digunakan untuk melakukan penjumlahan alamat/index.

Contoh:

```text
IX = 3000H
Offset = 05H
```

Maka:

\[
3000H+05H=3005H
\]

Alamat `3005H` kemudian digunakan pada **Address Bus**.

Secara konsep:

```text
IX
 │
 │ + Offset
 ↓
Adder
 │
 ↓
Address Bus 16-bit
```

Ini merupakan bagian penting dari **indexed addressing**.

---

## 11. Interrupt Logic

Bagian kiri atas adalah **Interrupt Logic**.

Pada gambar terdapat:

```text
IRQ0
IRQ1
IRQ2
```

IRQ berarti **Interrupt Request**.

Interrupt digunakan ketika perangkat eksternal ingin meminta perhatian CPU.

Contoh:

```text
Sensor / Peripheral
        │
        │ IRQ
        ↓
 Interrupt Logic
        │
        ↓
       CPU
```

Daripada CPU terus-menerus melakukan polling:

```text
Apakah ada data?
Apakah ada data?
Apakah ada data?
...
```

perangkat dapat memberikan interrupt:

> "CPU, saya membutuhkan perhatian."

CPU kemudian dapat menghentikan sementara program utama dan menjalankan **Interrupt Service Routine (ISR)**.

---

## 12. IFF — Interrupt Flip-Flop

Pada gambar terdapat bagian `IFF`.

IFF berkaitan dengan **Interrupt Flip-Flop**.

Secara sederhana:

```text
IFF = 1
→ interrupt enabled

IFF = 0
→ interrupt disabled
```

IFF digunakan untuk mengontrol apakah interrupt tertentu dapat diterima CPU.

---

## 13. INTA — Interrupt Acknowledge

**INTA** berarti:

> **Interrupt Acknowledge**

Ketika CPU menerima permintaan interrupt, CPU memberikan sinyal acknowledgment.

Secara konsep:

```text
Peripheral
    │
    │ IRQ
    ↓
Interrupt Logic
    │
    │ INTA
    ↓
Peripheral
```

Artinya CPU memberikan tanda bahwa permintaan interrupt telah diterima.

---

## 14. Blok `±1`

Bagian `±1` berkaitan dengan operasi **increment/decrement**.

Contoh:

```text
PC = 1000H
```

Kemudian:

```text
PC + 1 = 1001H
```

atau operasi pengurangan:

```text
SP - 1
```

Operasi increment/decrement sering diperlukan ketika CPU:

- mengambil instruksi
- memproses alamat
- mengakses stack
- memperbarui pointer

---

## 15. Hubungan Register dengan Data Bus

Banyak register terhubung ke **Data Bus 8-bit**.

Misalnya instruksi:

```asm
LD A,B
```

Secara konseptual:

```text
       B
       │
       ↓
Data Bus 8-bit
       │
       ↓
       A
```

Untuk pasangan register:

```text
B + C
  ↓
BC = 16 bit
```

CPU mengatur perpindahan high-byte dan low-byte sesuai instruksi yang sedang dijalankan.

---

## 16. Hubungan Accumulator, ALU, dan Flags

Ini merupakan salah satu bagian paling penting.

Misalnya:

```asm
ADD A,B
```

Secara sederhana:

```text
        B
        │
        ↓
       ┌───────┐
A ───→ │  ALU  │
       └───────┘
          │
          ├────→ A
          │
          └────→ F
```

Misalnya:

```text
A = 5
B = 3
```

ALU melakukan:

\[
5+3=8
\]

Kemudian:

```text
A = 8
```

Sementara `F` diperbarui berdasarkan hasil operasi.

---

## 17. Mengapa Address Bus 16-bit tetapi Data Bus 8-bit?

Ini merupakan konsep penting dalam memahami CPU.

CPU dapat memiliki:

```text
Data Bus    = 8 bit
Address Bus = 16 bit
```

Artinya CPU memproses data dalam unit **8-bit**, tetapi dapat memilih alamat memory menggunakan **16-bit**.

Contoh:

```text
Address:
1010 0011 0101 1100
        ↓
       16 bit
```

Memory kemudian mengirim data:

```text
11010110
   ↓
  8 bit
```

Jadi:

```text
CPU
 │
 ├── Address Bus → 16 bit → menentukan lokasi
 │
 └── Data Bus    → 8 bit  → membawa data
```

---

## 18. Contoh Proses CPU Secara Keseluruhan

Misalnya CPU menjalankan:

```asm
LD A,10
ADD A,20
```

### Langkah 1 — PC

PC memberikan alamat instruksi ke Address Bus:

```text
PC
 ↓
Address Bus
 ↓
Memory
```

### Langkah 2 — Fetch

Memory mengirim instruksi melalui Data Bus:

```text
Memory
   ↓
Data Bus 8-bit
   ↓
CPU
```

### Langkah 3 — Decode

CPU memahami instruksi:

```text
LD A,10
```

### Langkah 4 — Register A

```text
A = 10
```

### Langkah 5 — ALU

Ketika menjalankan:

```asm
ADD A,20
```

ALU melakukan:

```text
A = 10
Data = 20

       ┌─────┐
10 ──→ │ ALU │ ←── 20
       └─────┘
          │
          ↓
         30
```

### Langkah 6 — Register dan Flags

```text
A = 30
F = status hasil operasi
```

---

## 19. Gambaran Keseluruhan Arsitektur

Secara sederhana, diagram dapat dirangkum menjadi:

```text
                     CPU
                      │
       ┌──────────────┴──────────────┐
       │                             │
 Data Bus 8-bit              Address Bus 16-bit
       │                             │
       │                             │
 ┌─────┴──────┐                 ┌────┴─────┐
 │ Registers  │                 │ PC / SP  │
 │ A B C D E  │                 │ IX / IY  │
 │ H L        │                 └──────────┘
 └─────┬──────┘
       │
       ↓
     ┌─────┐
     │ ALU │
     └──┬──┘
        │
        ↓
     Flags
        │
        ↓
 Interrupt Logic
```

---

## 20. Ringkasan Fungsi Setiap Blok

| Blok | Ukuran | Fungsi |
|---|---:|---|
| `A` | 8 bit | Accumulator, register utama ALU |
| `F` | 8 bit | Menyimpan status/flags ALU |
| `B, C` | 8 + 8 bit | Register umum / pasangan BC 16-bit |
| `D, E` | 8 + 8 bit | Register umum / pasangan DE 16-bit |
| `H, L` | 8 + 8 bit | Register umum / pasangan HL 16-bit |
| `PC` | 16 bit | Menunjukkan alamat instruksi berikutnya |
| `SP` | 16 bit | Menunjukkan posisi stack |
| `IX` | 16 bit | Index register |
| `IY` | 16 bit | Index register |
| `ALU` | 8 bit | Operasi aritmetika dan logika |
| `Flags` | 8 bit | Status hasil operasi ALU |
| `Data Bus` | 8 bit | Membawa data |
| `Address Bus` | 16 bit | Membawa alamat |
| `Interrupt Logic` | — | Mengatur interrupt |
| `IFF` | — | Mengontrol enable/disable interrupt |
| `INTA` | — | Interrupt acknowledge |
| `Adder` | 16 bit | Perhitungan alamat, terutama IX/IY |

---

## Kesimpulan

Arsitektur pada gambar menunjukkan CPU dengan inti pemrosesan 8-bit dan kemampuan addressing 16-bit.

Alur utamanya dapat dipahami sebagai:

```text
              FETCH
                ↓
        PC → Address Bus
                ↓
             Memory
                ↓
        Data Bus 8-bit
                ↓
             DECODE
                ↓
           Register / ALU
                ↓
            EXECUTE
                ↓
          ALU + Flags
                ↓
       Register / Memory
```

**Inti pemahamannya:**

- **PC** menentukan *instruksi mana yang akan diambil*.
- **Address Bus** menentukan *lokasi memory*.
- **Data Bus** membawa *data/instruksi*.
- **Register** menyimpan data sementara.
- **ALU** melakukan operasi.
- **F/Flags** menyimpan status hasil operasi.
- **SP** menangani stack.
- **IX/IY** digunakan untuk indexed addressing.
- **Interrupt Logic** menangani permintaan dari perangkat eksternal.
