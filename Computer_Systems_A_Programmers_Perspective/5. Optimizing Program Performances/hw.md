## 5.13
Suppose we wish to write a procedure that computes the inner product of two vectors u and v. An abstract version of the function has a CPE of 14−18 with x86-64 for different types of integer and floating-point data. By doing the same sort of transformations we did to transform the abstract program `combine1` into the more efficient `combine4`, we get the following code:

```

1	/* Inner product. Accumulate in temporary */
2	void inner4(vec_ptr u, vec_ptr v, data_t *dest)
3	{
4		long i;
5		long length = vec_length(u);
6		data_t *udata = get_vec_start(u);
7		data_t *vdata = get_vec_start(v);
8		data_t sum = (data_t) 0;
9	
10		for (i = 0; i < length; i++) {
11			sum = sum + udata[i] * vdata[i];
12		}
13		*dest = sum;
14	}
```

Our measurements show that this function has CPEs of 1.50 for integer data and 3.00 for floating-point data. For data type double, the x86-64 assembly code for the inner loop is as follows:

```

	Inner loop of inner4. data_t = double, OP = *
	udata in %rbp, vdata in %rax, sum in %xmm0
	i in %rcx, limit in %rbx
1	.L15:					loop:
2	 vmovsd 0(%rbp,%rcx,8), %xmml		  Get udata[i]
3	 vmulsd (%rax,%rcx,8), %xmml, %xmml	  Multiply by vdata[i]
4	 vaddsd %xmml, %xmm0, %xmm0		  Add to sum
5	 addq $1, 7,rcx				  Increment i
6	 cmpq %rbx, %rcx			  Compare i:limit
7	 jne .L15				  If ! =, goto loop
```

Assume that the functional units have the characteristics listed in [Figure 5.12](https://reader.960960.xyz/fileP7000497027000000000000000004B6C.xhtml#P7000497027000000000000000004BAA).

1.  Diagram how this instruction sequence would be decoded into operations and show how the data dependencies between them would create a critical path of operations, in the style of [Figures 5.13](https://reader.960960.xyz/fileP7000497027000000000000000004B6C.xhtml#P7000497027000000000000000004C1E) and [5.14](https://reader.960960.xyz/fileP7000497027000000000000000004B6C.xhtml#P7000497027000000000000000004C33).
    
2.  For data type double, what lower bound on the CPE is determined by the critical path?
    
3.  Assuming similar instruction sequences for the integer code as well, what lower bound on the CPE is determined by the critical path for integer data?
    
4.  Explain how the floating-point versions can have CPEs of 3.00, even though the multiplication operation requires 5 clock cycles.


A) 
```c
       udata[i]
     /
OP1 --->
     \
       *vdata[i]
     /
OP2 --->
     \
       xmml
     /
OP3 --->
     \
       sum
     /
OP4 --->
     \
       sum
```
In the diagram, `OP1` represents the operation of fetching the value of `udata[i]`, `OP2` represents the operation of fetching the value of `vdata[i]`, `OP3` represents the operation of multiplying the two values, and `OP4` represents the operation of adding the result to the accumulated sum.

The critical path of operations is the longest sequence of operations that must be completed before the next iteration of the loop can begin. The critical path for the inner loop of `inner4` is:

`OP1 -> OP2 -> OP3 -> OP4`

This critical path is determined by the data dependencies between the operations, which are represented by the arrows in the diagram.

B) 
For data type double, the lower bound on the CPE is determined by the critical path, which requires at least 5 clock cycles to complete (the latency of the multiplication operation). Therefore, the lower bound on the CPE for double data is 5.

C) 
For integer data, we can assume that the instruction sequence is similar to the floating-point code, but with different instructions to perform the multiplication and addition operations. The lower bound on the CPE for integer data is also determined by the critical path, which requires at least the latency of the multiplication and addition operations.

D) 
The floating-point versions can have CPEs of 3.00, even though the multiplication operation requires 5 clock cycles, because the functional units in the processor may be able to perform multiple operations in parallel. For example, the processor may have multiple floating-point units, allowing it to perform multiple floating-point operations simultaneously. Additionally, the processor may be able to hide the latency of the multiplication operation by performing other operations in parallel with it. This allows the processor to keep its functional units busy, reducing the average number of clock cycles required to complete the critical path.

Therefore, the actual CPEs for the floating-point and integer data may be higher or lower than the lower bounds determined by the critical path, depending on the performance characteristics of the processor and the specific code being executed.

## 5.14
Write a version of the inner product procedure described in [Problem 5.13](https://reader.960960.xyz/#P7000497027000000000000000005092) that uses 6 × 1 loop unrolling. For x86-64, our measurements of the unrolled version give a CPE of 1.07 for integer data but still 3.01 for both floating-point data.

1.  Explain why any (scalar) version of an inner product procedure running on an Intel Core i7 Haswell processor cannot achieve a CPE less than 1.00.
    
2.  Explain why the performance for floating-point data did not improve with loop unrolling


Here is a version of the `inner4` function that uses 6 × 1 loop unrolling:
```c
void inner6(vec_ptr u, vec_ptr v, data_t *dest)
{
    long i;
    long length = vec_length(u);
    long limit = length - 5;
    data_t *udata = get_vec_start(u);
    data_t *vdata = get_vec_start(v);
    data_t sum = (data_t) 0;

    for (i = 0; i < limit; i += 6) {
        sum = sum + udata[i] * vdata[i] +
              udata[i+1] * vdata[i+1] +
              udata[i+2] * vdata[i+2] +
              udata[i+3] * vdata[i+3] +
              udata[i+4] * vdata[i+4] +
              udata[i+5] * vdata[i+5];
    }

    for (; i < length; i++) {
        sum = sum + udata[i] * vdata[i];
    }

    *dest = sum;
}
```

1) Any scalar version of an inner product procedure running on an Intel Core i7 Haswell processor cannot achieve a CPE less than 1.00 because the processor has a single floating-point unit and it takes at least one clock cycle to perform a floating-point operation. Since the inner product procedure involves multiple floating-point operations, it cannot be performed in less than one clock cycle.

2)  The performance for floating-point data did not improve with loop unrolling because the performance bottleneck is still the floating-point unit, and loop unrolling cannot change the performance of the floating-point unit. The loop unrolling only allows the processor to perform multiple floating-point operations in parallel, reducing the number of clock cycles required to complete each iteration of the loop. However, it still takes at least one clock cycle to perform each floating-point operation, and this remains the limiting factor for the performance of the inner product procedure.