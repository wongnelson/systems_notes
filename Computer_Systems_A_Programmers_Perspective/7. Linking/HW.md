##  7.8
In this problem, let `REF(x.i) → DEF(x.k)` denote that the linker will associate an arbitrary reference to symbol `x` in module `i` to the definition of `x` in module `k`. For each example below, use this notation to indicate how the linker would resolve references to the multiply-defined symbol in each module. If there is a link-time error (rule 1), write "`error`". If the linker arbitrarily chooses one of the definitions (rule 3), write "`unknown`".

1.  ```
    
    /* Module 1 */		/* Module 2 */
    int main()		static int main=1[
    {			int p2()
    }			{
    			}
    (a) REF(main.1) → DEF(_____._____)
    (b) REF(main.2) → DEF(_____._____)
```
```

(a) REF(main.1) → DEF(main.1) In Module 1, the symbol `main` is defined, so any references to `main` in Module 1 should be associated with this definition.

(b) REF(main.2) → DEF(main.2)
The linker's behavior in this case is determined by its rules for resolving multiply-defined symbols.

In this case, there are two modules that contain symbols with the same name (`main`). According to the rules:

1.  If there are multiple definitions of a symbol in different modules, and none of the definitions is marked as weak, then the linker will report a link-time error. This rule does not apply in this case, since there is only one definition of `main` in each module.
    
2.  If there are multiple definitions of a symbol in different modules, and one of the definitions is marked as weak, then the linker will choose the non-weak definition. This rule does not apply in this case, since there are no weak symbols involved.
    
3.  If there are multiple definitions of a symbol in different modules, and none of the definitions is marked as weak, then the linker will arbitrarily choose one of the definitions to use. This rule applies in this case, since there are multiple definitions of `main` with equal precedence, and neither is marked as weak.
    

Therefore, when Module 1 and Module 2 are linked together, the linker will choose one of the two definitions of `main` to use, and associate any references to `main` in both modules with that definition. Since there is a definition of `main` in both modules, the linker will choose the definition in the same module as the reference, so REF(main.2) → DEF(main.2) in Module 2.

b
```
/* Module 1 */		/* Module 2 */
int x;			double x;
void main()		int p2()
{			{
}			}
(a) REF(x.1) → DEF(_____._____)
(b) REF(x.2) → DEF(_____._____)
```
In Module 1, the symbol `x` is defined as an `int`, and in Module 2, the symbol `x` is declared as a `double`. Since there is no definition of `x` in Module 2, according to the linker's rules, any references to `x` in Module 2 will be associated with an arbitrary definition of `x` in another module. Since there is no other module with a definition of `x` that matches the type of `x` in Module 2, the linker would have to choose an arbitrary definition, which would result in unknown behavior at run time.

Therefore, the answers to both (a) REF(x.1) → DEF(unknown) and (b) REF(x.2) → DEF(unknown) are "unknown".

c
```
/* Module 1 */		/* Module 2 */
int x=1;		double x=1.0;
void main()		int p2()
{			{
}			}
(a) REF(x.1) → DEF(_____._____)
(b) REF(x.2) → DEF(_____._____)
```
The answer to both (a) and (b) is "error" because there are multiple definitions of `x` with the same name and type. This violates the one definition rule (ODR) in C++ and results in a link-time error.

In Module 1, `x` is defined as an `int`, and in Module 2, `x` is defined as a `double`. Even though the values assigned to the definitions are the same, the type of the definitions are different, so they are not compatible with each other.

Since the linker cannot resolve the ambiguity between the two definitions of `x`, it generates a link-time error, and the program cannot be successfully linked.


## 7.9
Consider the following program, which consists of two object modules:

```c

1	/* foo6.c */
2	void p2(void);
3
4	int main()
5	{
6		p2();
7		return 0;
8	}
```

```c

1	/* bar6.c */
2	#include <stdio.h>
3
4	char main;
5
6	void p2()
7	{
8		printf("0x%xn", main);
9	}
```

When this program is compiled and executed on an x86-64 Linux system, it prints the string `0x48\n` and terminates normally, even though function `p2` never initializes variable `main`. Can you explain this?

a; Upon compilation and execution of the given program on an x86-64 Linux system, it surprisingly prints the string `0x48\n` even though the variable `main` in the object module `bar6.c` is never initialized in the function `p2`.

<mark style="background: #FFB8EBA6;">As per the C standard, the linker should not allow the use of uninitialized variables.</mark> However, in this case, the linker has not only allowed the use of an uninitialized variable but also initialized it to a specific value.

This happens because of how the x86-64 Linux system initializes the data segment in executable programs. During program loading, the linker places the uninitialized global variables in a special section of the executable file called the "BSS" (Block Started by Symbol) section. The BSS section contains data that is uninitialized at the start of the program execution. This section is zeroed out by the operating system loader when the program is loaded into memory. So, when the program is run, the BSS section is zeroed out, and all uninitialized global variables are set to zero. In this case, the variable `main` is set to zero at program startup.

However, the `char` type of the variable `main` in `bar6.c` is a signed character, and when `printf` interprets the value of `main` as an `int`, it gets sign-extended to a negative value (in this case, `0xFFFFFF48`). This negative value, when printed in hex format with the `%x` format specifier, results in the output `0x48`.

Thus, even though the variable `main` is not explicitly initialized, it is initialized to zero due to the behavior of the BSS section. When the variable is used by the `printf` function, it is sign-extended to a negative value, resulting in the output `0x48\n`.

NOTE: READ THE ONLINE SOL


<mark style="background: #FFB8EBA6;">pink</mark>
According to the C standard, the use of uninitialized variables leads to undefined behavior. The C language does not specify what the initial value of a variable should be, so when the program uses an uninitialized variable, it is accessing a value that could be anything. Depending on the compiler and the platform, the behavior of the program could be unpredictable and could result in unexpected crashes or security vulnerabilities.

To prevent this, the C standard requires that the linker ensures that all uninitialized variables are initialized to a known value before they are used. This known value could be zero, or a default value, depending on the type of the variable. By ensuring that all variables have a known value before they are used, the C standard helps to prevent the occurrence of undefined behavior, and makes C programs more reliable and secure.

## 7.10
Let `a` and `b` denote object modules or static libraries in the current directory, and let `a`→`b` denote that `a` depends on `b`, in the sense that `b` defines a symbol that is referenced by `a`. For each of the following scenarios, show the minimal command line (i.e., one with the least number of object file and library arguments) that will allow the static linker to resolve all symbol references:

1.  ```
    p.o → libx.a → p.o
	In this scenario, we have only one object file and one static library. The object file `p.o` depends on symbols defined in `libx.a`. Therefore, we only need to link `p.o` with `libx.a` using the `gcc` command:
	gcc p.o libx.a

2.  ```
    p.o → libx.a → liby.a and liby.a → libx.a
    B. In this scenario, we have one object file and two static libraries, `libx.a` and `liby.a`. The object file `p.o` depends on symbols defined in `libx.a` and `liby.a`. Additionally, `liby.a` depends on symbols defined in `libx.a`. Therefore, we need to link `p.o`, `libx.a`, and `liby.a` in a specific order. Since `liby.a` depends on symbols defined in `libx.a`, we need to include `libx.a` before `liby.a` on the command line. Here's the command:
    `gcc p.o libx.a liby.a libx.a`
3.  ```
    p.o → libx.a → liby.a → libz.a and liby.a → libx.a → libz.a
	In this scenario, we have one object file and three static libraries, `libx.a`, `liby.a`, and `libz.a`. The object file `p.o` depends on symbols defined in `libx.a`, `liby.a`, and `libz.a`. Additionally, `liby.a` depends on symbols defined in `libx.a` and `libz.a`, and `libz.a` depends on symbols defined in `libx.a`. Therefore, we need to link `p.o`, `libx.a`, `liby.a`, and `libz.a` in a specific order. Since `liby.a` depends on symbols defined in `libx.a` and `libz.a`, and `libz.a` depends on symbols defined in `libx.a`, we need to include `libx.a` before `liby.a` and `libz.a` on the command line. Here's the command:
 `gcc p.o libx.a liby.a libx.a libz.a`