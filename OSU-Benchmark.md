a# Building OSU Micro-Benchmarks from Source on LANTA HPC
## 📌 Overview
[OSU Benchmarks (OSU)](https://mvapich.cse.ohio-state.edu/benchmarks) เป็นชุดเครื่องมือทดสอบประสิทธิภาพ MPI และ OpenSHMEM ที่นิยมใช้วัดค่าความหน่วง (latency), แบนด์วิดท์ (bandwidth) และประสิทธิภาพการสื่อสารของระบบ HPC เช่น Point-to-point MPI benchmarks และ One-sided MPI benchmarks

ในทุกระบบ HPC ผู้ใช้สามารถ build จาก source เพื่อใช้งานกับ MPI เวอร์ชันที่ต้องการได้ โดยใน README.md นี้จะสอนเกี่ยวกับการติดตั้ง Software ด้วยวิธีการใช้ bash script ซึ่งวิธีการนี้สามารถนำไปใช้ได้กับทุก HPC เพียงแค่เปลี่ยน library สำหรับ Software ที่ใช้

## 📚 สิ่งที่จำเป็นต้องรู้ เกี่ยวกับ Compiler และการ Compile Program บน HPC

บนระบบ HPC เช่น LANTA, GADI, ASPIRE-2A และอื่น ๆ การ compile โปรแกรมมีความแตกต่างจากบนคอมพิวเตอร์ส่วนบุคคล เพราะต้องพิจารณาเรื่องสถาปัตยกรรมของเครื่อง (Computer Architecture), ระบบจัดการ job (Job Scheduler), และการใช้โมดูล (modules) ให้ถูกต้อง

### 1. Compiler
- **GNU Compiler (GCC)** — เปิดกว้าง, ฟรี, ใช้งานทั่วไป, รองรับ MPI/OpenMP
  - ตัวอย่าง:  
    ```bash
    module load gcc
    ```
  - คำสั่งเรียก compiler:
    - `gcc` (สำหรับภาษา C)
    - `g++` (สำหรับภาษา C++)
    - `gfortran` (สำหรับภาษา Fortran)
- **Intel Compiler (ICC/ICPC/IFORT)** — ถูกออกแบบมาสำหรับ CPU ที่เป็น Intel ทำให้บางงานอาจคำนวณเร็วขึ้น
  - ตัวอย่าง:  
    ```bash
    module load intel
    ```
  - คำสั่งเรียก compiler:
    - `icc` (สำหรับภาษา C)
    - `icpc` (สำหรับภาษา C++)
    - `ifort` (สำหรับภาษา Fortran)

### 2. MPI Compiler Wrappers
เมื่อโหลดโมดูล MPI แล้ว เช่น OpenMPI หรือ Intel MPI จะมี wrapper ให้ใช้งาน:

ตัวอย่างเมื่อทำการ `module load OpenMPI`
- `mpicc` (สำหรับภาษา C)
- `mpicxx` หรือ `mpiCC` (สำหรับภาษา C++)
- `mpif90` หรือ `mpifort` (สำหรับภาษา Fortran)
Wrapper จะตั้ง include paths และ link libraries ให้อัตโนมัติ

หรือตัวอย่างเมื่อทำการ `module load Intel-mpi` (ใน LANTA ไม่มี แต่ใน GADI มี)
- `mpiicc` (สำหรับภาษา C)
- `mpiicpc` (สำหรับภาษา C++)
- `mpiifort` (สำหรับภาษา Fortran)
Wrapper จะตั้ง include paths และ link libraries ให้อัตโนมัติ

### 3. การ Compile โปรแกรมให้เหมาะกับ HPC
- ใช้ compiler และ MPI เดียวกัน (เช่น GNU Compiler + OpenMPI หรือ Intel Compiler + Intel MPI หรือบางครั้งใช้ Intel Compiler กับ OpenMPI)
- เพิ่ม flag ปรับแต่งประสิทธิภาพ เช่น:
  - GCC: `-O3 -march=native --fast-math`
  - Intel: `-O3 -xHost`
- ตรวจสอบสถาปัตยกรรมของ CPU ได้ด้วย `lscpu` ก่อนเพื่อตรวจสอบและเลือก flag ที่เหมาะสม

### 4. ข้อควรระวัง
- อย่า compile ในโฟลเดอร์ home ที่เต็มเร็ว ควรใช้พื้นที่ `/scratch` หรือ /project สำหรับงานขนาดใหญ่
- หากต้อง compile โปรแกรมขนาดใหญ่ ควรใช้ interactive job หรือ build script บน login node เฉพาะกรณีที่ใช้เวลาไม่นาน
- เก็บ binary ที่ compile แล้วแยกโฟลเดอร์ตาม MPI/Compiler เพื่อป้องกันการสับสน

### 5. ตัวอย่างการ Compile MPI Program
โหลด Software Compiler ที่ต้องการ
```bash
module restore
module load gcc
```
คำสั่งเรียกใช้ Compiler เพื่อ Compile โปรแกรมให้เหมาะสมกับสถาปัตยกรรมของเครื่อง
```bash
mpicc -O3 -march=native myprogram.c -o myprogram
```
---

## 🛠 วิธีการ Build ที่พบบ่อย

### 1. ใช้ `configure` + `make`

ใช้ในซอฟต์แวร์ที่รองรับ GNU Autotools ซึ่งจะมีไฟล์ชื่อ configure และ Makefile โดยภายใน Makefile จะประกอบไปด้วย configurations สำหรับ compile และ links dependencies ที่ต้องการ
ขั้นตอนทั่วไปสำหรับใช้ bash script เช่น:

```bash
#!/bin/bash

./configure --prefix=/path/to/install
make -j 4
make install
```

* `configure` ใช้ตรวจสอบระบบ, ค้นหา dependencies และเตรียมไฟล์ Makefile
* `--prefix` ใช้กำหนด path สำหรับติดตั้ง
* `-j 4` เป็นตัวเลือกเพื่อให้ `make` รันงานแบบ parallel โดยใช้ 4 threads เพื่อให้ compile เร็วขึ้น
* `make install` ใช้ติดตั้งไปยัง path ที่กำหนด

เรายังสามารถดู option ของไฟล์ `configure` ได้ด้วยคำสั่ง
```bash
./configure --help
```

---

### 2. ใช้ `cmake` + `make`

ใช้ในซอฟต์แวร์ที่ใช้ CMake เป็น build system ซึ่งจะมีไฟล์ชื่อ CMakeLists และ โฟลเดอร์ชื่อ Cmake ขั้นตอนทั่วไป เช่น:

```bash
#!/bin/bash

mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/path/to/install
make -j 4
make install
```

* `mkdir build` สร้างโฟลเดอร์ เพื่อแยกไฟล์ build ออกจาก source code
* `cmake` เป็นเครื่องมือสำหรับ generate ไฟล์ build (เช่น Makefile หรือ Ninja)
* สามารถกำหนด options ได้ด้วย -D เช่น `-DCMAKE_INSTALL_PREFIX` สำหรับตำแหน่งติดตั้ง
* `-DCMAKE_INSTALL_PREFIX` ใช้กำหนด path สำหรับติดตั้ง
* สามารถดู option ต่าง ๆ สำหรับ Software นั้นได้ที่ website เกี่ยวกับ Installation เช่น https://hoomd-blue.readthedocs.io/en/v5.3.1/building.html#configure

---

### 3. ใช้ `make` โดยตรง

บางซอฟต์แวร์ไม่มีขั้นตอน `configure` หรือ `cmake`

```bash
#!/bin/bash

make
make install
```

> มักใช้ใน Software ที่มี Makefile กำหนดการคอมไพล์ไว้แล้ว

---

### ก่อนการ build ควรจะใช้ `module load` เพื่อโหลด Dependencies ที่ต้องการ

บน HPC มักต้องโหลด module ของ compiler และ library ก่อน เช่น:

```bash
module load gcc/12.2.0
module load openmpi/4.1.5
```

> เพื่อให้มั่นใจว่าใช้ compiler และ MPI เวอร์ชันที่ถูกต้อง และเป็น version ที่ต้องการ ควรตั้งค่า environment ก่อนเรียก configure เพราะถ้าไม่ได้ set environment ไว้ Autotools อาจค้นหา dependencies ที่มีอยู่ในระบบ ซึ่งอาจไม่ตรงกับที่ต้องการ

## การติดตั้งโปรแกรม OSU Benchmark ด้วย GNU Compiler และ OpenMPI

## 📥 Step 1 — Download Source Code
โหลดโปรแกรม OSU Benchmark จาก source ในรูปแบบ tar file
```bash
wget https://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-7.5.1.tar.gz
```
แตก tar file ด้วยคำสั่ง
```bash
tar -xvf osu-micro-benchmarks-7.5.1.tar.gz
```
เข้าสู่โฟลเดอร์โปรแกรม
```bash
cd osu-micro-benchmarks-7.5.1
```

## 📦 Step 2 — อ่าน README และดู Software ที่ใช้ Build
> **ควรอ่าน README ทุกครั้งเมื่อจะทำการติดตั้งซอฟต์แวร์**

ในหลาย ๆ โปรแกรม จะมีไฟล์ `README`, `INSTALL` หรือ `USER_GUIDE` ที่อธิบายเกี่ยวกับ:
- รายละเอียดของซอฟต์แวร์
- Dependencies ที่ต้องการ
- ขั้นตอนการคอมไพล์และติดตั้ง
- ตัวเลือก (options) ที่สามารถใช้ในการ compile

ตัวอย่างการเปิดไฟล์ README:
```bash
less README
```

หรือ:

```bash
cat README
```
💡 **Tip**: การอ่าน README ก่อน จะช่วยให้การติดตั้งเป็นไปอย่างราบรื่น และหลีกเลี่ยงปัญหาที่เกิดจากการขาด dependency หรือใช้ compiler/module ไม่ถูกต้อง

---
## ⚙ Step 3 — สร้าง Bash Script

ก่อนสร้างไฟล์ `build.sh` ควรตรวจสอบให้แน่ใจว่า **มีการสร้างโฟลเดอร์ `app` ขึ้นมาหรือยัง** โดยโฟลเดอร์ `app` จะอยู่ที่ `$HOME/app`
หากยังไม่มี สามารถสร้างด้วยคำสั่ง:

```bash
mkdir -p $HOME/app
```

สร้างไฟล์เปล่าชื่อ `build.sh` ด้วยคำสั่ง:

```bash
touch build.sh
```

เปิดไฟล์ `build.sh` ด้วย text editor เช่น `nano` หรือ `vim` แล้วใส่เนื้อหาตามนี้:

```bash
#!/bin/bash

# โหลด module GCC compiler และ OpenMPI
module load gcc
module load OpenMPI

# รัน configure พร้อมกำหนด MPI compiler wrapper และ path ติดตั้ง
./configure CC=mpicc FC=mpifort CXX=mpicxx \
--prefix=$HOME/app/osu-benchmark

# คอมไพล์ด้วย make พร้อมใช้ 4 threads
make -j 4

# ติดตั้งโปรแกรมลงใน path ที่กำหนด
make install
```

---

### 📖 คำอธิบายโค้ดใน `build.sh`

1. **`#!/bin/bash`**

   * Shebang line เพื่อบอกระบบว่า script นี้ให้รันด้วย Bash shell

2. **`module load gcc` และ `module load OpenMPI`**

   * โหลด GCC compiler และ MPI module จาก environment ของ HPC
   * จำเป็นเพราะ HPC บางระบบไม่ได้ติดตั้ง GCC ใน path เริ่มต้น

3. **`./configure CC=mpicc FC=mpifort CXX=mpicxx \`**  
   - รันสคริปต์ `configure` เพื่อเตรียมไฟล์ Makefile สำหรับการคอมไพล์  
   - **`CC`** คือ environment variable ที่กำหนดว่าโปรแกรมจะใช้ **C compiler** ตัวใดในการคอมไพล์โค้ดภาษา C  
     - ที่นี่ตั้งเป็น `mpicc` ซึ่งเป็น **MPI C compiler wrapper** ที่ช่วยตั้งค่า compiler flags และ include paths สำหรับ MPI อัตโนมัติ  
   - **`FC`** คือ environment variable สำหรับ **Fortran compiler** โดยตั้งเป็น `mpifort` (MPI Fortran wrapper)  
   - **`CXX`** คือ environment variable สำหรับ **C++ compiler** โดยตั้งเป็น `mpicxx` (MPI C++ wrapper)

4. **`--prefix=$HOME/app/osu-benchmark`**

   * ระบุ directory ปลายทางที่โปรแกรมจะถูกติดตั้ง
   * ตัวอย่างนี้จะติดตั้งไว้ที่ `$HOME/app/osu-benchmark`

5. **`make -j 4`**

   * คอมไพล์โปรแกรมโดยใช้ 4 threads เพื่อให้เร็วขึ้น
   * ตัวเลข 4 สามารถปรับตามจำนวน CPU cores ที่มี

6. **`make install`**

   * ติดตั้งไฟล์ที่คอมไพล์เสร็จแล้วไปยัง path ที่กำหนดใน `--prefix`

---
## ▶️ Step 4 — การรัน Bash Script

หลังจากสร้างและแก้ไขไฟล์ `build.sh` เรียบร้อยแล้ว  
เราสามารถรันสคริปต์เพื่อทำขั้นตอนการ build และติดตั้งโปรแกรมได้

### วิธีการรัน Bash Script

1. **รันด้วยคำสั่ง `bash` หรือ `sh`**  
   ```bash
   bash build.sh
   ```

   หรือ
   ```bash
   sh build.sh
   ```

* รันสคริปต์ใน shell ใหม่ (subshell)
* การเปลี่ยนแปลง environment ภายในสคริปต์จะไม่ส่งผลต่อ shell ปัจจุบัน

2. **รันด้วยคำสั่ง `.` (dot command) หรือ `source`**

   ```bash
   . build.sh
   ```

   หรือ

   ```bash
   source build.sh
   ```

   * รันสคริปต์ใน shell ปัจจุบัน
   * การเปลี่ยนแปลง environment เช่น การโหลด module หรือการตั้งค่า environment variables จะส่งผลใน shell ปัจจุบันทันที
   * เหมาะสำหรับกรณีที่ต้องการโหลด module หรือเซ็ต environment สำหรับคำสั่งถัดไป

---
### หมายเหตุ
* ถ้าต้องการรันสคริปต์โดยตรง สามารถตั้งสิทธิ์ execute แล้วรันได้เลย

  ```bash
  chmod +x build.sh
  ./build.sh
  ```
* แต่การรันด้วย `.` หรือ `source` จะเหมาะกับการโหลด module หรือ environment variables ที่ต้องใช้ต่อใน shell ปัจจุบัน

---
## Step 5 — ตรวจสอบการ build software และ run simple benchmark
หลังจากติดตั้งโปรแกรมเสร็จสิ้นแล้ว ควรตรวจสอบว่าโปรแกรมถูกติดตั้งและทำงานได้อย่างถูกต้อง โดยการรันโปรแกรมตัวอย่างหรือ benchmark แบบง่าย ๆ
### 1. ตรวจสอบไฟล์ Binary ที่ติดตั้ง  
ตรวจสอบว่าไฟล์ executable ถูกติดตั้งใน path ที่กำหนดไว้ (`$HOME/app/osu-benchmark/bin` หรือโฟลเดอร์ที่คุณระบุ)


```bash
tree $HOME/app/osu-benchmark/
```
