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

ตัวอย่างเมื่อทำการ module load OpenMPI
- `mpicc` (สำหรับภาษา C)
- `mpicxx` หรือ `mpiCC` (สำหรับภาษา C++)
- `mpif90` หรือ `mpifort` (สำหรับภาษา Fortran) 
Wrapper จะตั้ง include paths และ link libraries ให้อัตโนมัติ

หรือตัวอย่างเมื่อทำการ module load Intel-mpi
- `mpiicc` (สำหรับภาษา C)
- `mpiicpc` (สำหรับภาษา C++)
- `mpiifort` (สำหรับภาษา Fortran)  
Wrapper จะตั้ง include paths และ link libraries ให้อัตโนมัติ

### 3. การ Compile โปรแกรมให้เหมาะกับ HPC
- ใช้ compiler และ MPI เดียวกัน (เช่น GNU Compiler + OpenMPI หรือ Intel Compiler + Intel MPI หรือบางครั้งใช้ Intel Compiler กับ OpenMPI)
- เพิ่ม flag ปรับแต่งประสิทธิภาพ เช่น:
  - GCC: `-O3 -march=native`
  - Intel: `-O3 -xHost`
- ตรวจสอบสถาปัตยกรรม CPU ด้วย `lscpu` ก่อนเพื่อเลือกและตรวจสอบ flag ที่เหมาะสม

### 4. ข้อควรระวัง
- อย่า compile ในโฟลเดอร์ home ที่เต็มเร็ว ควรใช้พื้นที่ `/scratch` หรือ /project สำหรับงานขนาดใหญ่
- หากต้อง compile โปรแกรมขนาดใหญ่ ควรใช้ Slurm interactive job หรือ compile บน login node เฉพาะกรณีที่ใช้เวลาไม่นาน
- เก็บ binary ที่ compile แล้วแยกตาม MPI/Compiler เพื่อป้องกันการสับสน

### 5. ตัวอย่างการ Compile MPI Program
```bash
module purge
module load compiler/gnu/11.3.0
module load mpi/openmpi/4.1.5

mpicc -O3 -march=native myprogram.c -o myprogram

---

## 📥 Step 1 — Download Source Code

```bash
wget https://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-7.5.1.tar.gz
```
```bash
tar -xvf osu-micro-benchmarks-7.5.1.tar.gz
```
```bash
cd osu-micro-benchmarks-7.5.1
```

## 📦 Step 2 — Load MPI Module

ตัวอย่างการโหลด OpenMPI 4.1.5:
```bash
module purge
module load gcc
module load mpi/openmpi/4.1.5
```

💡 สำหรับ Intel MPI:
```bash
module load compiler/intel mpi/impi/2021
```

## ⚙ Step 3 — Configure Build
สร้างโฟลเดอร์ติดตั้ง:

```bash
mkdir -p $HOME/osu-benchmarks
./configure CC=mpicc CXX=mpicxx --prefix=$HOME/osu-benchmarks
```

หมายเหตุ:

CC=mpicc และ CXX=mpicxx ใช้ MPI compiler wrapper

--prefix กำหนดตำแหน่งติดตั้ง (เช่น $HOME หรือ scratch)
