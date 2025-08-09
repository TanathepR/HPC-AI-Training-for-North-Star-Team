# Building OSU Micro-Benchmarks from Source on LANTA HPC
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

## 📦 Step 2 — อ่าน README  
> **ควรอ่าน README ทุกครั้งเมื่อจะทำการติดตั้งซอฟต์แวร์**

ในหลาย ๆ โปรแกรม จะมีไฟล์ `README`, `INSTALL` หรือ `USER_GUIDE` ที่อธิบายเกี่ยวกับ:
- รายละเอียดของซอฟต์แวร์
- Dependencies ที่ต้องติดตั้งก่อน
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
💡 **Tip**: การอ่าน README ก่อน จะช่วยให้การติดตั้งเป็นไปอย่างราบรื่น
และหลีกเลี่ยงปัญหาที่เกิดจากการขาด dependency หรือใช้ compiler/module ไม่ถูกต้อง

## ⚙ Step 3 — Configure Build
สร้างโฟลเดอร์ติดตั้ง:

```bash
mkdir -p $HOME/osu-benchmarks
./configure CC=mpicc CXX=mpicxx --prefix=$HOME/osu-benchmarks
```

หมายเหตุ:

CC=mpicc และ CXX=mpicxx ใช้ MPI compiler wrapper

--prefix กำหนดตำแหน่งติดตั้ง (เช่น $HOME หรือ scratch)
