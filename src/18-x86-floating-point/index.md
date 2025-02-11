# Вещественные числа на x86

## FPU (x87)

[8 80-битных регистров (extended precision)](https://www.club155.ru/x86internalreg-fpucommon)

[FLD — загрузить число из памяти в стек FPU](https://www.felixcloutier.com/x86/fld)

[FST/FSTP — сохранить вершину стека FPU в память](https://www.felixcloutier.com/x86/fst:fstp)

[Пример операции: FADD](https://www.felixcloutier.com/x86/fadd:faddp:fiadd)

## Регистры SSE

SSE (Streaming SIMD Extension) - набор инструкций, позволяющий выполнять несколько одинаковых
операций одновременно. Набор инструкций SSE продолжает расширяться.

Для хранения аргументов операций SSE используются регистры `xmm0 ... xmm15`. Регистры xmm являются scratch-регистрами, то есть при вызове подпрограмм
сохранение значений не гарантируется.

Регистры xmm имеют размер 128 бит и могут хранить 2 64-битных, 4 32-битных целых или вещественных значения,
а также 8 16-битных или 16 8-битных целых значения. Интерпретация битового содержимого регистров xmm
зависит от выполняемой инструкции.

В стандартном соглашении о вызовах x64 первые 8 параметров вещественных типов
float или double передаются на регистрах `xmm0 ... xmm7`, последующие аргументы
передаются в стеке. Результат вещественного типа возвращается в регистре `xmm0`.


## Скалярные вычисления на регистрах SSE

Регистры SSE можно использовать для обычных вычислений с плавающей точкой. Такие инструкции по терминологии
Intel называются скалярными. В этом случае в регистрах xmm будет использоваться
только младшая часть: младшие 32 или 64 бита.

Для пересылки скалярных значений могут использоваться следующие инструкции:
```
        movsd   SRC, DST        // пересылка между регистрами xmm и памятью значения double
        movss   SRC, DST        // пересылка значения типа float
```
Эти инструкции позволяют пересылать значение из регистра xmm в другой регистр xmm, а также между регистрами xmm и памятью.
Выравнивание на размер типа повышает производительность.

Со скалярными значениями поддерживаются, например, следующие операции:
```
        addsd   SRC, DST        // DST += SRC, double
        addss   SRC, DST        // DST += SRC, float
        subsd   SRC, DST        // DST -= SRC, double
        subss   SRC, DST        // DST -= SRC, float
        mulsd   SRC, DST        // DST *= SRC, double
        mulss   SRC, DST        // DST *= SRC, float
        divsd   SRC, DST        // DST /= SRC, double
        divss   SRC, DST        // DST /= SRC, float
        sqrtsd  SRC, DST        // DST = sqrt(SRC), double
        sqrtss  SRC, DST        // DST = sqrt(SRC), float
        maxsd   SRC, DST        // DST = max(SRC, DST), double
        maxss   SRC, DST        // DST = max(SRC, DST), float
        minsd   SRC, DST        // DST = min(SRC, DST), double
        minss   SRC, DST        // DST = min(SRC, DST), float
```

Преобразование double->int выполняется инструкцией
```
        cvtsd2si SRC, DST       // DST = (int32_t) SRC
```
Здесь SRC - регистр xmm или память, DST - 32-битный регистр общего назначения.
Инструкция выполняет преобразование вещественног числа типа double в 32-битное знаковое целое число.

Преобразование double->float выполняется инструкцией:
```
        cvtsd2ss SRC, DST       // DST = (float) SRC
```

Преобразование int->double выполняется инструкцией:
```
        cvtsi2sd SRC, DST       // DST должен быть регистр xmm, SRC либо GPR, либо память
```

Преобразование float->double:
```
        cvtss2sd SRC, DST       // DST = (double) SRC
```

Для преобразований float->int и int->float предназначены инструкции cvtss2si и cvtsi2ss.

Сравнение двух скалярных значений типа float или double выполняется инструкцией:
```
        comisd  SRC, DST        // DST - SRC, double
        comiss  SRC, DST        // DST - SRC, float
```
В результате выполнения операции сравнения устанавливаются флаги PF, CF, ZF. Флаг PF устанавливается,
если результат - неупорядочен. Флаг ZF устанавливается, если значения равны.
Флаг CF устанавливается, если DST < SRC. Для условного перехода после сравнения можно
использовать условные переходы для беззнаковых чисел. Например, ja будет выполнять условный переход,
если DST > SRC.

## Векторные вычисления на регистрах SSE

Векторные вычисления в терминологии Intel описываются как вычисления с упакованными (packed) значениями.

Для пересылки 128-битных значений между памятью и регистрами xmm и между двумя регистрами xmm
используется инструкция
```
        movapd  SRC, DST        // DST = SRC
```
если один из аргументов - память, адрес должен быть выровнен по адресу, кратному 16.
Для пересылки по невыровненным адресам можно использовать инструкцию movupd.

С векторными значениями поддерживаются следующие операции, которые выполняются
одновременно со всеми значениями в регистрах (2 для double или 4 для float):
```
        addpd   SRC, DST        // DST += SRC, double
        addps   SRC, DST        // DST += SRC, float
        subpd   SRC, DST        // DST -= SRC, double
        subps   SRC, DST        // DST -= SRC, float
        mulpd   SRC, DST        // DST *= SRC, double
        mulps   SRC, DST        // DST *= SRC, float
        divpd   SRC, DST        // DST /= SRC, double
        divps   SRC, DST        // DST /= SRC, float
        sqrtpd  SRC, DST        // DST = sqrt(SRC), double
        sqrtps  SRC, DST        // DST = sqrt(SRC), float
        maxpd   SRC, DST        // DST = max(SRC, DST), double
        maxps   SRC, DST        // DST = max(SRC, DST), float
        minpd   SRC, DST        // DST = min(SRC, DST), double
        minps   SRC, DST        // DST = min(SRC, DST), float
```

## Горизонтальные операции

Обычная операция над упакованными SSE-регистрами может рассматриваться как "вертикальная". Например,
рассмотрим инструкцию `ADDPS A, B`. Эта инструкция складывает четыре float-значения в операнде A
с соответствующими 4 значениями в операнде B и кладет результат в операнд B. Если A и B рассматривать
как массивы из 4 значений типа float, то операция может быть описана следующим образом:

```
    float A[4];
    float B[4];

    B[0] = A[0] + B[0]
    B[1] = A[1] + B[1]
    B[2] = A[2] + B[2]
    B[3] = A[3] + B[3]
```

В противовес "вертикальной" операции "горизонтальная" операция вовлекает соседние значение в одном регистре.
Например, инструкция `HADDPS A, B` выполняется следующим образом:

```
    float A[4];
    float B[4];

    B[0] = B[0] + B[1];
    B[1] = B[2] + B[3];
    B[2] = A[0] + A[1];
    B[3] = A[2] + A[3];
```

## AVX
[AVX](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions)

[Применение масок](https://habr.com/ru/companies/intel/articles/266055/)

[SIMDJSON](https://branchfree.org/2019/02/25/paper-parsing-gigabytes-of-json-per-second/)

## Обработка ошибок

[Floating point environment](https://en.cppreference.com/w/c/numeric/fenv)

[Always use feenableexcept](https://berthub.eu/articles/posts/always-do-this-floating-point/)

## Оптимизация

[Опция `-ffast-math`](https://kristerw.github.io/2021/10/19/fast-math/)