Milvus的计算操作依赖于CPU对SIMD（Single Instruction, Multiple Data）扩展指令集的支持。您的CPU是否支持SIMD扩展指令集对于Milvus中的索引构建和向量相似度搜索至关重要。请确保您的CPU支持以下至少一种SIMD指令集：

- SSE4.2
- AVX
- AVX2
- AVX512

运行以下命令（lscpu）来检查您的CPU是否支持上述SIMD指令集：

```
$ lscpu | grep -e sse4_2 -e avx -e avx2 -e avx512
```