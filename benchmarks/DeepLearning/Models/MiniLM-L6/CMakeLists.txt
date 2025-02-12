# Compile MLIR file to object file.
add_custom_command(OUTPUT minilm-l6-default.o
COMMAND 
  ${LLVM_MLIR_BINARY_DIR}/mlir-opt ${CMAKE_CURRENT_SOURCE_DIR}/MiniLM-L6-200.mlir
      --linalg-bufferize
      --convert-linalg-to-loops
      --func-bufferize
      --arith-bufferize
      --tensor-bufferize
      --finalizing-bufferize
      --convert-vector-to-scf
      --convert-scf-to-cf
      --expand-strided-metadata
      --lower-affine
      --convert-vector-to-llvm
      --memref-expand
      --arith-expand
      --convert-arith-to-llvm
      --finalize-memref-to-llvm
      --convert-math-to-llvm
      --llvm-request-c-wrappers
      --convert-func-to-llvm
      --reconcile-unrealized-casts | 
  ${LLVM_MLIR_BINARY_DIR}/mlir-translate --mlir-to-llvmir |
  ${LLVM_MLIR_BINARY_DIR}/llc -mtriple=${BUDDY_OPT_TRIPLE} -mattr=${BUDDY_OPT_ATTR} 
    --filetype=obj -relocation-model=pic -o ${CMAKE_CURRENT_BINARY_DIR}/minilm-l6-default.o
)

add_library(MiniLMDefault STATIC minilm-l6-default.o)
set_target_properties(MiniLMDefault PROPERTIES LINKER_LANGUAGE CXX)

add_executable(minilm-l6-benchmark Main.cpp MiniLMDefaultBenchmark.cpp)
# Link libraries
target_link_directories(minilm-l6-benchmark PRIVATE ${LLVM_MLIR_LIBRARY_DIR})
target_link_libraries(minilm-l6-benchmark
  MiniLMDefault
  GoogleBenchmark
  mlir_c_runner_utils
)
