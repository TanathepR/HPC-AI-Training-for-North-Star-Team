# Building OSU Micro-Benchmarks from Source on LANTA HPC
## 📌 Overview
[OSU Benchmarks (OSU)](https://mvapich.cse.ohio-state.edu/benchmarks) เป็นชุดเครื่องมือทดสอบประสิทธิภาพ MPI และ OpenSHMEM ที่นิยมใช้วัดค่าความหน่วง (latency), แบนด์วิดท์ (bandwidth) และประสิทธิภาพการสื่อสารของระบบ HPC เช่น Point-to-point MPI benchmarks และ One-sided MPI benchmarks

ในทุกระบบ HPC ผู้ใช้สามารถ build จาก source เพื่อใช้งานกับ MPI เวอร์ชันที่ต้องการได้ โดยใน README.md นี้จะสอนเกี่ยวกับการติดตั้ง Software ด้วยวิธีการใช้ bash script ซึ่งวิธีการนี้สามารถนำไปใช้ได้กับทุก HPC เพียงแค่เปลี่ยน library สำหรับ Software ที่ใช้

## 📚 สิ่งที่จำเป็นต้องรู้ เกี่ยวกับ Compiler และการ Compile Program บน HPC

บนระบบ HPC เช่น LANTA การ compile โปรแกรมมีความแตกต่างจากบนคอมพิวเตอร์ส่วนบุคคล เพราะต้องพิจารณาเรื่องสถาปัตยกรรมของเครื่อง, ระบบจัดการ job, และการใช้โมดูล (modules) ให้ถูกต้อง

### 1. Compiler Modules
- **GNU Compiler (GCC)** — เปิดกว้าง, ฟรี, ใช้งานทั่วไป, รองรับ MPI/OpenMP
  - ตัวอย่าง: `module load compiler/gnu/11.3.0`
- **Intel Compiler (ICC/ICPC/IFORT)** — ปรับแต่งให้เหมาะกับสถาปัตยกรรม Intel ทำให้บางงานวิ่งเร็วขึ้น
  - ตัวอย่าง: `module load compiler/intel`
- **MPI Compiler Wrappers** — คำสั่งอย่าง `mpicc`, `mpicxx`, `mpif90` จะใช้ compiler ที่เหมาะกับ MPI เวอร์ชันที่โหลด

### 2. MPI Compiler Wrappers
เมื่อโหลดโมดูล MPI แล้ว เช่น OpenMPI หรือ Intel MPI จะมี wrapper ให้ใช้งาน:
- `mpicc` → สำหรับภาษา C
- `mpicxx` หรือ `mpiCC` → สำหรับภาษา C++
- `mpif90` หรือ `mpifort` → สำหรับ Fortran  
Wrapper จะตั้ง include paths และ link libraries ให้อัตโนมัติ

### 3. การ Compile ให้เหมาะกับ HPC
- ใช้ compiler และ MPI จากโมดูลเดียวกัน (เช่น GCC + OpenMPI หรือ Intel Compiler + Intel MPI)
- เพิ่ม flag ปรับแต่งประสิทธิภาพ เช่น:
  - GCC: `-O3 -march=native`
  - Intel: `-O3 -xHost`
- ตรวจสอบสถาปัตยกรรม CPU ด้วย `lscpu` ก่อนเพื่อเลือก flag ที่เหมาะสม

### 4. ข้อควรระวัง
- อย่า compile ในโฟลเดอร์ home ที่เต็มเร็ว ควรใช้พื้นที่ `/scratch` สำหรับงานใหญ่
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
