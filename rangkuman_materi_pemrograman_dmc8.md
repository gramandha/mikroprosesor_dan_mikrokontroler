# Pemrograman DMC8 — Rangkuman Materi

---

## 3.1 Pengantar Pemrograman Bahasa Assembly

### 3.1.1 Bahasa Pemrograman
* **Bahasa Tingkat Tinggi (Basic, C, C++, Java):** Mudah dipahami manusia, tetapi satu konstruksi dapat menghasilkan banyak instruksi mesin. Diterjemahkan menggunakan *compiler*.
* **Bahasa Tingkat Rendah (Assembly):** Terdiri dari instruksi *mnemonic* yang berkorespondensi satu-satu dengan instruksi mesin. Diterjemahkan menggunakan *assembler*.
* **Interpreter:** Menerjemahkan kode sumber secara langsung baris demi baris saat eksekusi berlangsung.

#### Alur Pemrograman:
```text
Source Program → Translator (Compiler/Assembler) → Object Code → Linker → Programmer → ROM → Microprocessor-based System
```
* **Linker:** Menggabungkan beberapa modul objek menjadi satu kode mesin eksekutabel.
* **Loader/Programmer:** *Programmer* (perangkat keras) memuat kode mesin ke ROM secara permanen, sedangkan *loader* memuatnya ke memori sistem.

---

### 3.1.2 Bahasa Assembly DMC8
Setiap baris kode dalam bahasa assembly DMC8 umumnya terdiri dari 4 kolom (*field*):

| Field | Keterangan |
| :--- | :--- |
| **Label** | Referensi untuk lokasi/lompatan program (opsional). |
| **Mnemonic** | Kode *mnemonic* instruksi CPU. |
| **Operand** | Parameter atau data operasi untuk instruksi. |
| **Comment** | Catatan/komentar (opsional), diawali dengan titik koma `;`. |

#### Contoh Program Penjumlahan 8-bit:
```assembly
SUM:  LD A, (0500h)   ; muat operand pertama
      LD B, A
      LD A, (0501h)   ; muat operand kedua
      ADD A, B        ; jumlahkan kedua operand
      LD (8000h), A   ; simpan hasil ke RAM
      HALT
```
> **Catatan:** Assembler tidak bersifat *case-sensitive* (`sum:` sama dengan `SUM:`).

---

### 3.1.3 Konstanta dan Variabel
* Bahasa tingkat tinggi mendukung deklarasi tipe data untuk konstanta dan variabel, sedangkan bahasa assembly umumnya tidak.
* *Programmer* menentukan sendiri alokasi alamat memori dan jumlah byte yang digunakan.
* **Contoh:** Operand disimpan di ROM alamat `0500h` dan `0501h`; hasil simpanan diletakkan di RAM alamat `8000h`.

---

### 3.1.4 Direktif EQU
`EQU` (*equal*) merupakan sebuah direktif (*pseudo-instruction*), yaitu perintah khusus untuk *assembler* dan bukan instruksi mesin CPU.

```assembly
RESULT EQU 8000h      ; RESULT = alamat 8000h
```

* **Keuntungan:** Apabila lokasi alamat berubah, *programmer* cukup mengubah satu baris definisi simbol.

#### Aturan Sintaks Penting:
1. Nama simbol harus diawali dengan huruf.
2. Bilangan heksadesimal yang diawali huruf harus diberi angka `0` di depannya (contoh: `0FEBAH` adalah bilangan, sedangkan `FEBAH` dianggap sebagai simbol).

#### Dua Tahapan (*Two-Pass*) Assembler:
* **Pass 1:** Mengidentifikasi alamat label dan simbol, lalu menyimpannya ke dalam *symbol table*.
* **Pass 2:** Mengganti simbol/label dengan nilainya pada kode mesin.

---

### 3.1.5 Direktif ORG
`ORG` (*origin*) menentukan alamat awal pengalokasian kode mesin di memori.

```assembly
        ORG 0000h
        JP 0100h      ; lompat ke program utama

        ORG 0100h
SUM:    LD A, (0500h)
        ...
```
* **`0000h`:** Lokasi *reset* prosesor.
* **`0008h` – `0038h`:** Alamat yang dicadangkan untuk *interrupt handler*.
* **Taktik Klasik:** Melakukan lompatan (`JP 0100h`) untuk melewati area memori yang dicadangkan.

---

### 3.1.6 Direktif DB dan DW
* **`DB` (Define Byte):** Mendefinisikan konstanta berukuran 8-bit (1 byte).
* **`DW` (Define Word):** Mendefinisikan konstanta berukuran 16-bit (2 byte).

#### Contoh Penggunaan DB:
```assembly
AVALUE EQU 3Fh
        ORG 0800h
CONST1: DB AVALUE              ; simpan 3Fh di 0800h
BYTES:  DB 0,1,2,3,4,5,6,7     ; 8 byte mulai alamat 0801h
ASCII:  DB "ABCDEF"            ; 6 kode ASCII
DIGITS: DB "0123456789"        ; 10 karakter ASCII
        DB "A""B""C"           ; tanda kutip ganda = karakter "
NUMBERS: DB 8,-8,-58,AVALUE    ; 4 konstanta
MIX:    DB "XY","ZW",0,0FFh    ; data campuran
```
> **Aturan Bilangan Negatif:** Dikodekan dalam bentuk *two's complement* 8-bit (rentang $-128$ hingga $-1$). Nilai $-1$, $255$, dan `FFh` menghasilkan representasi bit yang sama.

#### Contoh Penggunaan DW:
```assembly
WORDS: DW 8,-8,1033,AVALUE     ; 4 konstanta 16-bit
       DW 0,0,0,0,0,0,0,0      ; 16 byte bernilai nol
```
> **Konvensi Little-Endian:** Byte rendah (*low byte*) disimpan di alamat memori yang lebih rendah, sedangkan byte tinggi (*high byte*) disimpan di alamat berikutnya.

---

## 3.2 Mode Pengalamatan (Addressing Modes)

Mode pengalamatan menentukan cara CPU mengambil operand atau data untuk suatu instruksi. DMC8 memiliki 9 mode pengalamatan:

1. **IMMEDIATE** (data 8-bit)
2. **EXTENDED IMMEDIATE** (data 16-bit)
3. **DIRECT** (alamat memori)
4. **REGISTER INDIRECT**
5. **INDEXED INDIRECT**
6. **REGISTER**
7. **IMPLIED**
8. **BIT**
9. **MODIFIED**

> Tidak semua kombinasi mode diizinkan. Contoh: tidak ada instruksi `LD (3500h), (1F00h)` — transfer antar memori harus melalui register (misalnya register `A`).

---

### 3.2.1 IMMEDIATE (8-bit)
Operand sebesar 8-bit terletak tepat setelah *opcode*.

```assembly
LD A, 00h    ; Opcode: 3Eh 00h
LD B, 5Fh    ; Opcode: 06h 5Fh
```

### 3.2.2 EXTENDED IMMEDIATE (16-bit)
Operand sebesar 16-bit (2 byte) terletak setelah *opcode*.

```assembly
LD HL, 0C035h   ; Opcode: 21h 35h C0h (35h → L, C0h → H)
LD SP, 0FF00h
LD BC, 0FAFFh
```

### 3.2.3 DIRECT
Alamat lokasi memori dicantumkan langsung setelah *opcode*.

* **Data 8-bit:**
  ```assembly
  LD (0E000h), A   ; 32h 00h E0h
  IN A, (20h)      ; DBh 20h
  ```
  *(Tanda kurung `()` menunjukkan akses terhadap isi dari alamat tersebut).*

* **Data 16-bit:**
  ```assembly
  LD HL, (820Fh)   ; 2Ah 0Fh 82h (Byte pertama → L, Byte kedua → H)
  ```

### 3.2.4 REGISTER INDIRECT
Alamat data diambil dari nilai yang tersimpan di register 16-bit (`BC`, `DE`, `HL`).

```assembly
LD C, (HL)       ; 4Eh (membaca 1 byte data)
```

#### Contoh Inisialisasi Tabel:
```assembly
        LD HL, 80F0h
        LD (HL), 3Fh
        INC HL
        LD (HL), 12h
        INC HL
```

#### Contoh Inisialisasi 50 Lokasi Memori dengan Nol:
```assembly
        LD HL, 0A700h
        LD B, 50
LOOP:   LD (HL), 00h
        INC HL
        DEC B
        JP NZ, LOOP
```
> Mode ini merupakan dasar penerapan konsep *pointer* pada bahasa tingkat tinggi.

### 3.2.5 INDEXED INDIRECT
Alamat memori = isi register indeks (`IX` atau `IY`) + *displacement* (konstanta $-128$ hingga $+127$).

```assembly
LD B, (IY+2Fh)   ; FDh 46h 2Fh
LD IX, 9000h
LD A, (IX+33h)
CP (IX+01h)
```
> Sangat berguna untuk mengakses elemen atau variabel di dalam struktur data.

### 3.2.6 REGISTER
Informasi register asal dan tujuan sudah terkandung di dalam *opcode*.

```assembly
LD A, D          ; Opcode 7Ah
CP B
AND L
```

### 3.2.7 IMPLIED
Operasi menggunakan register yang sudah ditentukan secara otomatis (*implisit*).

```assembly
ADD A, L         ; 85h (Register A secara implisit sebagai tujuan)
SCF              ; 37h (Set Carry Flag)
NOP              ; 00h (No Operation)
```

### 3.2.8 BIT
Digunakan oleh instruksi `RES`, `SET`, dan `BIT` untuk menentukan indeks bit tertentu ($0$–$7$).

```assembly
RES 2, B         ; CBh 9Eh
SET 3, A
SET 5, (HL)
BIT 0, E
```
> Indeks bit harus berupa konstanta, bukan variabel atau register.

### 3.2.9 MODIFIED
Khusus digunakan oleh instruksi `RST` (*Restart*) untuk melompat ke alamat-alamat standar (`0000h`, `0008h`, ..., `0038h`).

```assembly
RST 38h          ; Opcode FFh
```

---

## 3.3 Jenis-jenis Instruksi

Kategori instruksi DMC8 meliputi:
1. Transfer Data (*Load*)
2. Aritmetika & Logika
3. *Shift* & *Rotate*
4. Manipulasi Bit
5. Lompatan (*Jump*)
6. Call & Return Subprogram
7. Input/Output (I/O)
8. Kontrol CPU

---

### 3.3.1 Instruksi Transfer Data
**Format:** `LD <destination>, <source>`

* **Transfer 8-bit:**
  * **Sumber:** Nilai *immediate* (`LD A, 3Bh`), isi register (`LD A, B`), isi memori (`LD A, (0F001h)`, `LD C, (HL)`).
  * **Tujuan:** Register (`LD B, E`), lokasi memori (`LD (0F305h), A`, `LD (HL), A`).

* **Transfer 16-bit:**
  * Register yang didukung: `BC`, `DE`, `HL`, `SP`, `IX`, `IY`.
  ```assembly
  LD BC, 5F3Dh
  LD SP, HL
  LD HL, 0F140h
  LD IX, 8001h
  LD (0F140h), HL
  ```

* **Instruksi Stack:**
  ```assembly
  PUSH HL          ; Simpan HL ke stack
  POP IX           ; Ambil isi stack ke IX
  ```

---

### 3.3.2 Instruksi Aritmetika dan Logika
> **Catatan:** ALU DMC8 hanya mendukung operasi dasar. Tidak tersedia instruksi perkalian atau pembagian langsung secara hardware.

#### Aritmetika 8-bit
Register `A` selalu berperan sebagai operand pertama sekaligus penampung hasil operasi.

* **ADD:**
  ```assembly
  ADD A, 01h
  ADD A, B
  ADD A, (HL)
  ADD A, (IX+16)
  ADD A, (IY+32)
  ```

* **ADC (Add with Carry):** Menambahkan operand beserta nilai *Carry flag*.
  ```assembly
  ADC A, (HL)
  ```

  *Contoh Penjumlahan 16-bit (ADD + ADC):*
  ```assembly
  OPE_A  EQU 8000h
  OPE_B  EQU 8002h
  RESULT EQU 8004h

  ADD16:  LD BC, OPE_A
          LD HL, OPE_B
          LD DE, RESULT
          LD A, (BC)
          ADD A, (HL)
          LD (DE), A       ; Simpan byte rendah
          INC BC
          INC HL
          INC DE
          LD A, (BC)
          ADC A, (HL)      ; Simpan byte tinggi + Carry
          LD (DE), A
  ```

* **SUB & SBC:**
  ```assembly
  SUB 39h
  SUB E
  SUB (HL)
  SUB (IX+22)
  SUB (IY+44)
  ```

* **CP (Compare):** Melakukan pengurangan tanpa mengubah nilai register `A`, hanya memperbarui kondisi *flag*.
  ```assembly
  CP 75h
  CP B
  CP (HL)
  CP (IX+56)
  CP (IY+12)
  ```

  | Hasil Pembandingan | Carry (C) | Zero (Z) | Sign (S) |
  | :--- | :---: | :---: | :---: |
  | $A > s$ | 0 | 0 | 0 |
  | $A = s$ | 0 | 1 | 0 |
  | $A < s$ | 1 | 0 | 1 |

  *Contoh Penerapan CP:*
  ```assembly
  CHECK:  LD A, (VAR)
          CP 3Ah
          JP Z, EQUAL1
          CP 20h
          JP Z, EQUAL2
          JP NC, MAJOR
  MINOR:  ...
          JP CONTINUE
  MAJOR:  ...
  CONTINUE: ...
  ```

* **CPL dan NEG:**
  ```assembly
  LD A, (8000h)
  CPL              ; Komplemen Satu (inversi semua bit)
  ADD A, 1         ; +1
  NEG              ; Komplemen Dua (negasi nilai)
  ```

#### Aritmetika 16-bit
Menggunakan register `HL`, `IX`, atau `IY` sebagai akumulator.

```assembly
ADD HL, BC
ADD HL, HL       ; Menggandakan nilai HL (HL * 2)
ADD IX, DE
```

*Contoh Penjumlahan 16-bit Singkat:*
```assembly
ADD16:  LD BC, OPE_A
        LD HL, OPE_B
        ADD HL, BC
        LD (RESULT), HL
```

*Contoh Algoritma Penjumlahan 64-bit:*
```assembly
OPE_A  EQU 8000h
OPE_B  EQU 8008h
RESULT EQU 8010h
NBYTE  EQU 8

ADD64:  LD IX, OPE_A
        LD IY, OPE_B
        LD HL, RESULT
        LD A, (IX)
        ADD A, (IY)
        LD (HL), A
        LD B, NBYTE
        DEC B
ADDBYTE: INC IX
        INC IY
        INC HL
        LD A, (IX)
        ADC A, (IY)
        LD (HL), A
        DEC B
        JP NZ, ADDBYTE
```

#### Instruksi Logika
* **AND, OR, XOR:** Operasi bitwise dengan operand implisit Register `A`.
  ```assembly
  AND 01h
  AND B
  AND (HL)
  AND (IX+16)
  AND (IY+32)
  ```

* **Bitmasking dengan AND:**
  ```assembly
  LD A, (0F000h)
  AND 00000111b    ; Hanya mempertahankan bit 0, 1, dan 2
  JP Z, ZERO
  ```

* **Inversi Bit dengan XOR:**
  ```assembly
  LD A, 11110000b
  LOOP:   OUT (00h), A
          XOR 00001111b    ; Inversi bit 3..0
          JP LOOP
  ```

#### Increment/Decrement 8-bit
Mengubah nilai sebesar $+1$ atau $-1$. Memengaruhi *Zero* dan *Sign flag*, namun **tidak** memengaruhi *Carry flag*. Bersifat siklik ($255 + 1 = 0$).

```assembly
INC A
INC B
INC (HL)
DEC H
DEC (IX)
DEC (IY+3)
```

#### Increment/Decrement 16-bit
Digunakan pada `BC`, `DE`, `HL`, `IX`, `IY`. **Tidak memengaruhi flag apapun**.

```assembly
INC HL
DEC IX
```

*Contoh Inisialisasi RAM (Penggunaan OR untuk Cek Nol 16-bit):*
```assembly
MEM  EQU 8000h
NLOC EQU 2048

INIT_RAM: LD HL, MEM
          LD BC, NLOC
LOOP:     LD (HL), 00h
          INC HL
          DEC BC
          LD A, B
          OR C             ; Gabungkan B dan C untuk cek apakah BC == 0
          JP NZ, LOOP
```

---

### 3.3.3 Instruksi Rotate dan Shift
Operasi pergeseran bit pada operand 8-bit (register atau memori via `HL`/`IX`/`IY`).

| Instruksi | Fungsi |
| :--- | :--- |
| **RLC** | *Rotate Left Circular* (bit 7 dipindah ke Carry dan bit 0) |
| **RRC** | *Rotate Right Circular* |
| **RL** | *Rotate Left* 9-bit (Carry diposisikan sebagai bit ke-9) |
| **RR** | *Rotate Right* 9-bit |
| **SLA** | *Shift Left Arithmetic* (Sama dengan perkalian 2) |
| **SRL** | *Shift Right Logic* (Pergeseran logika ke kanan) |
| **SRA** | *Shift Right Arithmetic* (Sama dengan pembagian 2, mempertahankan bit tanda) |

#### Contoh RLC (Efek LED Berjalan):
```assembly
LD A, 00000001b
LOOP:   OUT (00h), A
        RLC A
        JP LOOP
```

#### Contoh SLA + RL untuk Rotasi 16-bit (Register Pair DE):
```assembly
LOOP:   LD A, D
        OUT (01h), A
        LD A, E
        OUT (00h), A
        SLA E
        RL D
        JP NC, LOOP
        SET 0, E
        JP LOOP
```
> **RLCA, RRCA, RLA, RRA:** Versi rotasi cepat khusus Register `A` (kompatibilitas Intel 8080) yang hanya memengaruhi *Carry flag*.

---

### 3.3.4 Instruksi Manipulasi Bit
* **`BIT`:** Menguji kondisi bit tertentu ($0$–$7$) dan memperbarui *Zero flag*.
* **`SET`:** Mengubah bit tertentu menjadi `1`.
* **`RES`:** Mengubah bit tertentu menjadi `0`.

```assembly
BIT 0, A
BIT 3, D
BIT 7, (HL)
RES 2, B
SET 7, A
```

#### Contoh Deteksi Transisi Sinyal 0 → 1:
```assembly
TESTO:  IN A, (00h)
        BIT 0, A
        JP NZ, TESTO
TEST1:  IN A, (00h)
        BIT 0, A
        JP Z, TEST1
        INC B
        RES 7, B
        LD A, B
        OUT (00h), A
        JP TESTO
```

---

### 3.3.5 Instruksi Lompatan (Jump)

#### Unconditional Jump
```assembly
JP 2000h      ; Lompat langsung ke alamat 2000h
JP LOOP       ; Lompat ke label LOOP
```

#### Conditional Jump
Format: `JP <kondisi>, <alamat>`

| Kondisi | Operasi | Kondisi Flag |
| :---: | :--- | :---: |
| **Z** | *Jump if zero* | $Z = 1$ |
| **NZ** | *Jump if not zero* | $Z = 0$ |
| **C** | *Jump if Carry* | $C = 1$ |
| **NC** | *Jump if not Carry* | $C = 0$ |
| **P** | *Jump if positive* | $S = 0$ |
| **M** | *Jump if minus/negative* | $S = 1$ |
| **PE** | *Jump if parity even* | $P = 1$ |
| **PO** | *Jump if parity odd* | $P = 0$ |

#### Indirect Jump & Jump Table
```assembly
JP (HL)
JP (IX)
JP (IY)
```

*Penerapan Jump Table:*
```assembly
JTABLE: DW FIRST
        DW SECOND
        DW THIRD
        DW FOURTH

        SLA A
        LD E, A
        LD D, 00h
        LD HL, JTABLE
        ADD HL, DE
        LD E, (HL)
        INC HL
        LD D, (HL)
        LD L, E
        LD H, D
        JP (HL)         ; Lompat ke alamat hasil pencarian tabel
```

#### Delay Loops (Loop Penundaan)
* **Delay Sederhana:**
  ```assembly
  LD C, 255
  LOOP:   DEC C
          JP NZ, LOOP
  ```
  *Perhitungan Siklus:*
  $$N = 7 + 255 \times (4 + 10) = 3577 \text{ siklus clock}$$
  Jika $F_{ck} = 10\text{ MHz}$ ($T_{ck} = 100\text{ ns}$):
  $$T_d = 3577 \times 100\text{ ns} = 0,3577\text{ ms}$$

  *Rumus Menghitung Counter:*
  $$X = \frac{N - 7}{22}$$

* **Nested Loop (Loop Bersarang):**
  ```assembly
          LD D, 255
  LEXT:   LD C, 255
  LINT:   DEC C
          JP NZ, LINT
          DEC D
          JP NZ, LEXT
  ```

* **Delay dengan Counter 16-bit:**
  ```assembly
          LD BC, 65535
  LOOP:   DEC BC
          LD A, B
          OR C
          JP NZ, LOOP
  ```

---

### 3.3.6 Instruksi Kontrol CPU
```assembly
EI       ; Enable Interrupts (IFF ← 1)
DI       ; Disable Interrupts (IFF ← 0)
NOP      ; No Operation (memakan 4 siklus clock)
HALT     ; Memhentikan eksekusi CPU hingga ada interrupt/reset
SCF      ; Set Carry Flag (C ← 1)
CCF      ; Complement Carry Flag (C ← NOT C)
OR A     ; Clear Carry Flag (C ← 0) tanpa merubah nilai register A
```

---

### 3.3.7 Instruksi Input/Output (I/O)
Kapasitas alamat port I/O maksimum mencakup 256 alamat input dan 256 alamat output (menggunakan bus alamat $A_7..A_0$).

```assembly
IN A, (20h)      ; Baca port input di alamat 20h ke register A
OUT (0FFh), A    ; Tulis nilai register A ke port output alamat FFh
```

#### Mode Indirect (Alamat Port di Register C):
```assembly
IN r, (C)        ; r = A, B, C, D, E, H, L
OUT (C), r
```
> Mode `IN r, (C)` memperbarui status *flag* CPU.

*Contoh Menyalin Data 8 Lokasi Memori ke 8 Port Output:*
```assembly
        LD HL, 8000h
        LD C, 00h
        LD B, 8
LOOP:   LD A, (HL)
        OUT (C), A
        INC HL
        INC C
        DEC B
        JP NZ, LOOP
```

---

## 3.4 Subprogram dan Area Stack

### 3.4.1 Stack dan Stack Pointer (SP)
* **Stack:** Area memori RAM yang digunakan untuk menyimpan dan mengambil data atau alamat sementara.
* **Stack Pointer (SP):** Register 16-bit yang menyimpan alamat puncak *stack*.
* *Stack* bertumbuh menuju alamat memori yang lebih rendah (*tumbuh ke bawah*).

```assembly
LD SP, 0FFFFh    ; Inisialisasi titik awal/dasar stack
```

#### Operasi PUSH dan POP
```assembly
PUSH BC
PUSH HL
...
POP HL
POP BC
```
* **Operand yang didukung:** `AF`, `BC`, `DE`, `HL`, `IX`, `IY`.
* **Aturan Utama:**
  1. Jumlah instruksi `PUSH` harus sama dengan jumlah `POP`.
  2. Urutan `POP` harus merupakan kebalikan (*mirror*) dari urutan `PUSH`.

---

### 3.4.2 Subprogram dan Instruksi Call/Return
* **Keuntungan Subprogram:** Menghemat pemakaian memori, struktur kode lebih rapi, dan modular (dapat digunakan berulang kali).

```assembly
CALL <alamat>
RET
```

#### Contoh Subprogram Menghitung Rata-rata:
```assembly
START:  LD SP, 0FFFFh
        LD B, 34
        LD C, 15
        CALL AVERAGE
        LD E, A

        LD B, 56
        LD C, 22
        CALL AVERAGE
        LD D, A
        ...

AVERAGE: LD A, B
        ADD A, C
        SRA A
        RET
```

#### Cara Kerja CALL dan RET pada Stack:
* **`CALL`:** Menyimpan alamat kembali (*return address* / isi register `PC`) ke dalam *stack*, lalu melompat ke alamat subprogram.
* **`RET`:** Mengambil *return address* dari puncak *stack* dan memasukkannya kembali ke register `PC`.

> **Mengapa tidak menggunakan JP?** Karena instruksi `JP` tidak menyimpan alamat kembali ke *stack*, sehingga subprogram tidak bisa kembali ke titik panggil semula secara dinamis.

#### Trik Khusus Penggunaan Stack:
* **Menukar Nilai Register Pair `BC` dan `DE` tanpa Register Akses:**
  ```assembly
  EXBCDE: PUSH BC
          PUSH DE
          POP BC
          POP DE
          RET
  ```

* **Mengeksekusi "JP (BC)" Melalui Stack:**
  ```assembly
  PUSH BC
  RET
  ```

#### Conditional CALL dan RET:
```assembly
DEC B
CALL Z, SUBPROG
LD D, E
```

---

## 3.5 Contoh-contoh Pemrograman

### 3.5.1 Emulasi Logika Kombinasional

#### Gerbang NOT
```assembly
INP  EQU 00h
OUTP EQU 00h

        ORG 0000h
        JP START

        ORG 0100h
START:  IN A, (INP)
        XOR 00000001b    ; Inversi bit 0
        OUT (OUTP), A
        JP START
```

#### Gerbang AND 2-Input
* **Teknik 1 — Uji Bit Satu per Satu:**
  ```assembly
  START:  IN A, (INP)
          BIT 0, A
          JP Z, OUT0
          BIT 1, A
          JP Z, OUT0
  OUT1:   LD A, 00000001b
          JP OUTPUT
  OUT0:   LD A, 00000000b
  OUTPUT: OUT (OUTP), A
          JP START
  ```

* **Teknik 2 — Bitmask:**
  ```assembly
  START:  IN A, (INP)
          XOR 0FFh
          AND 03h
          JP Z, OUT1
  OUT0:   LD A, 00000000b
          JP OUTPUT
  OUT1:   LD A, 00000001b
  OUTPUT: OUT (OUTP), A
          JP START
  ```

* **Teknik 3 — Shift + AND:**
  ```assembly
  START:  IN A, (INP)
          LD B, A
          SRL B
          AND B
          OUT (OUTP), A
          JP START
  ```

#### Multiplexer 2-ke-1
```assembly
START:  IN A, (INP)
        SRL A
        JP NC, OUTPUT
        SRL A
OUTPUT: OUT (OUTP), A
        JP START
```

#### Dekoder 3-ke-8

* **Metode Linear Decoding:**
  ```assembly
  START:  IN A, (INP)
          AND 00000111b
  TEST0:  CP 0
          JP NZ, TEST1
          LD A, 00000001b
          JP OUTPUT
  TEST1:  CP 1
          ...
  ```

* **Metode Decoding by Calculations:**
  ```assembly
  START:  LD B, 00000001b
          IN A, (INP)
          AND 00000111b
  LOOP:   JP Z, OUTPUT
          SLA B
          DEC A
          JP LOOP
  OUTPUT: LD A, B
          OUT (OUTP), A
          JP START
  ```

* **Metode Decoding by Tables:**
  ```assembly
  TABLE:  DB 00000001b
          DB 00000010b
          ...

  START:  IN A, (INP)
          AND 00000111b
          LD HL, TABLE
          ADD A, L
          LD L, A
          JP NC, OUTPUT
          INC H
  OUTPUT: LD A, (HL)
          OUT (OUTP), A
          JP START
  ```

---

### 3.5.2 Menghitung Polinomial
Hitung persamaan: $OA = 1.5 \cdot IA + 5.0 \cdot IB + 0.75 \cdot IC$

```assembly
; Subprogram Perkalian 1.5 (A * 1.5)
Mult_1.5: PUSH BC
          LD B, A
          SRA B            ; B = A / 2
          ADD A, B         ; A = A + A/2
          POP BC
          RET

; Subprogram Perkalian 5.0 (A * 5.0)
Mult_5.0: PUSH BC
          LD B, A
          SLA B
          SLA B            ; B = A * 4
          ADD A, B         ; A = A + A*4
          POP BC
          RET

; Subprogram Perkalian 0.75 (A * 0.75)
Mult_0.75: CALL Mult_1.5
           SRA A           ; (A * 1.5) / 2
           RET

; Program Utama Polinomial
POLY_ABC: IN A, (IAport)
          CALL Mult_1.5
          LD B, A
          IN A, (IBport)
          CALL Mult_5.0
          ADD A, B
          LD B, A
          IN A, (ICport)
          CALL Mult_0.75
          ADD A, B
          OUT (OAPort), A
          RET
```

---

### 3.5.3 Timer
```assembly
TRIGP  EQU 00h
PULSEP EQU 00h

        ORG 0000h
        JP START

        ORG 0100h
START:  LD SP, 0FFFFh
MAIN:   LD A, 00h
        OUT (PULSEP), A
CHECK:  IN A, (TRIGP)
        BIT 0, A
        JP NZ, CHECK
UPEDGE: IN A, (TRIGP)
        BIT 0, A
        JP Z, UPEDGE
        LD A, 00000001b
        OUT (PULSEP), A
        CALL DELAY
        JP MAIN

DELAY:  LD B, 10
ExtLoop: LD DE, 41667
IntLoop: DEC DE
        LD A, D
        OR E
        JP NZ, IntLoop
        DEC B
        JP NZ, ExtLoop
        RET
```

---

### 3.5.4 Finite State Machine (FSM)
```assembly
INP  EQU 00h
OUTP EQU 00h

        ORG 0000h
        JP START

        ORG 0100h
START:  LD SP, 0FFFFh
        IN A, (INP)
        LD E, A

STATE_A: LD A, 00000000b
        CALL CHECK
        JP Z, STATE_A

STATE_B: LD A, 00000001b
        CALL CHECK
        JP Z, STATE_A

STATE_C: LD A, 00000010b
        CALL CHECK
        JP Z, STATE_B

STATE_D: LD A, 00000011b
        CALL CHECK
        JP Z, STATE_C
        JP STATE_D

CHECK:  OUT (OUTP), A
LOOP:   LD A, E
        CPL
        LD B, A
        IN A, (INP)
        LD E, A
        AND B
        BIT 0, A
        JP Z, LOOP
        BIT 1, E
        RET
```

---

## 3.6 Latihan

### 3.6.1 Emulasi Komponen Digital
1. AND 8-input
2. OR 8-input
3. Jaringan AND-OR kombinasional
4. Shift register SIPO 8-bit
5. Shift register SIPO 16-bit
6. Counter biner sinkron 8-bit dengan *pre-load*
7. Counter up/down 12-bit siklik
8. Counter Gray code 4-bit
9. Counter BCD 4-digit

### 3.6.2 Fungsi Aritmetika
1. Rata-rata dua variabel 32-bit
2. Rata-rata tabel 256 nilai 8-bit
3. Perkalian dua bilangan 8-bit
4. Evaluasi fungsi $Y = \lfloor 127 \cdot \sin(X \cdot 360 / 256) \rfloor$

### 3.6.3 Modul dan Fungsi Reusable
1. Subprogram inisialisasi RAM
2. Rotasi konfigurasi bit pada port output
3. Display termometer 32 LED
4. LED berkedip dengan periode berbeda
5. Generator gelombang kotak
6. Transmisi serial (paralel ke serial)

---

## 3.7 Solusi (Ringkasan Kunci)

### 3.7.1 Emulasi Komponen Digital
* **AND 8-input:** Gunakan `CP 11111111b`
* **OR 8-input:** Gunakan `OR A`
* **AND-OR:** Dapat diimplementasikan dengan `CP` atau pembacaan tabel
* **SIPO 8-bit:** Gunakan `SRL B` dan `SET 7, B`
* **SIPO 16-bit:** Gunakan `SRL H` dan `RR L`
* **Counter 8-bit:** Gunakan `INC C`, `CP 0FFh`, lalu output Terminal Count (TC)
* **Counter 12-bit:** Gunakan `INC HL`/`DEC HL` dipadukan dengan mask `00001111b`
* **Gray Code:** Dihitung dengan rumus $G = Q \oplus (Q \gg 1)$
* **BCD Counter:** Gunakan register counter terpisah untuk unit, puluhan, ratusan, dan ribuan

### 3.7.2 Fungsi Aritmetika
* **MEAN32:** Penjumlahan bertahap menggunakan `ADC`, dilanjutkan pergeseran `SRA` + `RR`.
* **MEAN256:** Akumulasi nilai dengan *carry counter*; pembagian dengan 256 cukup dilakukan dengan mengambil byte tinggi (*high byte*).
* **MUL8BIT:** Menggunakan algoritma *long multiplication* dengan kombinasi `SRL D` dan `SLA C`/`RL B`.
* **SIN127:** Memanfaatkan *lookup table* 128 nilai, menggunakan `NEG` untuk memproses siklus paruh negatif.

### 3.7.3 Modul Reusable
* **WRAM:** Loop pengisian memori `LD (HL), A`; `INC HL`; `DEC C`.
* **Rotasi 7-bit:** Pergeseran `SLA`/`SRL` dengan `SET` bit untuk kondisi siklik.
* **Display Termometer:** Menggunakan lookup table 33 baris $\times$ 4 byte.
* **LED Berkedip:** Memanfaatkan counter biner 4-bit dan panggilan subprogram `OUTPUT`.
* **Gelombang Kotak:** Hitung siklus eksekusi instruksi dan tambahkan *delay loop*.
* **Transmisi Serial:** Kombinasi urutan `READ` + `SEND` dengan durasi *bit time* 0,1 ms.

---

## 4. Interfacing dengan Perangkat Eksternal

## 4.1 Handshake
*Handshake* adalah protokol sinkronisasi transfer data antar dua sistem terpisah.

### 4.1.1 Handshake Unidirectional
* **Strobe:** Pulsa validasi yang dikirimkan dari transmitter (Tx) ke receiver (Rx).
* Digunakan ketika kecepatan penerimaan data oleh Rx dijamin cukup cepat.

#### Contoh Program Tx:
```assembly
CTRLP EQU 00h
DATAP EQU 01h

MAIN:   CALL CREATE
        INC B
        LD A, B
        OUT (DATAP), A
        LD A, 00000001b  ; Aktifkan sinyal Strobe
        OUT (CTRLP), A
        CALL PTIME
        LD A, 00000000b  ; Nonaktifkan Strobe
        OUT (CTRLP), A
        JP MAIN
```

#### Contoh Program Rx:
```assembly
WAIT0:  IN A, (STATP)
        BIT 0, A
        JP NZ, WAIT0
WAIT1:  IN A, (STATP)
        BIT 0, A
        JP Z, WAIT1
        IN A, (DATAP)    ; Baca data setelah Strobe terdeteksi
        CALL PROCESS
        JP MAIN
```

---

### 4.1.2 Handshake Bidirectional
Menambahkan jalur sinyal **Busy** dari Rx menuju Tx.

* **Sisi Tx:** Menunggu hingga kondisi `Busy = 0` sebelum mengirimkan data baru:
  ```assembly
  WAIT:   IN A, (STATP)
          BIT 7, A
          JP NZ, WAIT    ; Tunggu jika Rx masih sibuk (Busy = 1)
          LD A, B
          OUT (DATAP), A
  ```

* **Sisi Rx:** Mengaktifkan indikator `Busy` saat memproses data:
  ```assembly
          LD A, 00000000b
          OUT (BUSYP), A
  WAIT0:  ...
  WAIT1:  ...
          LD A, 00000001b; Set status Busy = 1
          OUT (BUSYP), A
          IN A, (DATAP)  ; Ambil data
  ```

---

### 4.1.3 Handshake Lebih Kompleks
Menambahkan jalur sinyal **Ack** (*Acknowledge*) sebagai konfirmasi aktif dari penerima bahwa data telah berhasil dibaca.

---

## 4.2 Handshake dengan Dukungan Hardware
Menggunakan komponen *SR Flip-Flop* (dua gerbang NAND) untuk mengelola sinyal jabat tangan (*handshake*) secara otomatis tanpa membebani siklus CPU.

### Keuntungan:
1. Pulsa *Strobe* dapat dibuat sangat pendek.
2. Sinyal *Ready* dapat dipertahankan stabilitasnya oleh *hardware*.
3. Proses pembacaan dan penulisan data menjadi lebih andal dan efisien.