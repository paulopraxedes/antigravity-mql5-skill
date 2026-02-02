
---

## 📁 File 17: `skills/12_gpu_opencl.md`

```markdown
## SKILL-012: GPU Acceleration (OpenCL)

**Feature ID:** GPU-001  
**Domain:** High-Performance Computing  
**Scope:** OpenCL kernels, matrix operations, parallel processing

---

### 12.1 OpenCL Setup and Matrix Multiplication

```cpp
class CGPUAccelerator
{
private:
   int               m_context;
   int               m_program;
   int               m_kernel;
   bool              m_available;
   
public:
   CGPUAccelerator() : m_context(0), m_program(0), m_kernel(0), m_available(false) {}
   
   ~CGPUAccelerator() { Cleanup(); }
   
   bool Initialize()
   {
      //--- Check device availability
      int devices[];
      if(CLGetDevInfo(CL_DEVICE_TYPE_GPU, devices) == 0)
      {
         Print("No GPU devices available");
         return false;
      }
      
      //--- Create context
      m_context = CLContextCreate(devices[0]);
      if(m_context == INVALID_HANDLE) return false;
      
      m_available = true;
      Print("GPU acceleration initialized");
      return true;
   }
   
   bool LoadKernel(string kernel_code, string kernel_name)
   {
      if(!m_available) return false;
      
      //--- Create program
      m_program = CLProgramCreate(m_context, kernel_code);
      if(m_program == INVALID_HANDLE) return false;
      
      //--- Create kernel
      m_kernel = CLKernelCreate(m_program, kernel_name);
      
      return m_kernel != INVALID_HANDLE;
   }
   
   //--- Matrix multiplication: C = A × B
   bool MatrixMultiply(const matrix &A, const matrix &B, matrix &C)
   {
      if(!m_available) return false;
      if(A.Cols() != B.Rows()) return false;
      
      int m = A.Rows();
      int n = B.Cols();
      int k = A.Cols();
      
      C.Init(m, n);
      
      //--- Create buffers
      int bufferA = CLBufferCreate(m_context, m * k * sizeof(double));
      int bufferB = CLBufferCreate(m_context, k * n * sizeof(double));
      int bufferC = CLBufferCreate(m_context, m * n * sizeof(double));
      
      if(bufferA == INVALID_HANDLE || bufferB == INVALID_HANDLE || 
         bufferC == INVALID_HANDLE)
      {
         CleanupBuffers(bufferA, bufferB, bufferC);
         return false;
      }
      
      //--- Write data
      CLBufferWrite(bufferA, 0, A);
      CLBufferWrite(bufferB, 0, B);
      
      //--- Set kernel arguments
      CLSetKernelArg(m_kernel, 0, bufferA);
      CLSetKernelArg(m_kernel, 1, bufferB);
      CLSetKernelArg(m_kernel, 2, bufferC);
      CLSetKernelArg(m_kernel, 3, m);
      CLSetKernelArg(m_kernel, 4, n);
      CLSetKernelArg(m_kernel, 5, k);
      
      //--- Execute
      uint global_work_offset[2] = {0, 0};
      uint global_work_size[2] = {(uint)m, (uint)n};
      
      bool success = CLExecute(m_kernel, 2, global_work_offset, global_work_size);
      
      if(success)
      {
         CLBufferRead(bufferC, 0, C);
      }
      
      CleanupBuffers(bufferA, bufferB, bufferC);
      return success;
   }
   
   void Cleanup()
   {
      if(m_kernel != 0) CLKernelFree(m_kernel);
      if(m_program != 0) CLProgramFree(m_program);
      if(m_context != 0) CLContextFree(m_context);
      
      m_kernel = 0;
      m_program = 0;
      m_context = 0;
      m_available = false;
   }
   
private:
   void CleanupBuffers(int a, int b, int c)
   {
      if(a != INVALID_HANDLE) CLBufferFree(a);
      if(b != INVALID_HANDLE) CLBufferFree(b);
      if(c != INVALID_HANDLE) CLBufferFree(c);
   }
};

//--- OpenCL Kernel Code (string constant)
const string MATRIX_MULTIPLY_KERNEL = R"(
   __kernel void matrix_multiply(
      __global const double* A,
      __global const double* B,
      __global double* C,
      int M, int N, int K)
   {
      int row = get_global_id(0);
      int col = get_global_id(1);
      
      if(row < M && col < N)
      {
         double sum = 0.0;
         for(int i = 0; i < K; i++)
         {
            sum += A[row * K + i] * B[i * N + col];
         }
         C[row * N + col] = sum;
      }
   }
)";

---

### Parallel Indicator Calculation

```cpp
//--- GPU-accelerated moving average for large datasets
bool CalculateMA_GPU(const double &prices[], int period, double &ma[])
{
   CGPUAccelerator gpu;
   
   if(!gpu.Initialize()) 
      return false;  // Fallback to CPU
      
   if(!gpu.LoadKernel(MA_KERNEL, "moving_average"))
      return false;
   
   int n = ArraySize(prices);
   ArrayResize(ma, n);
   
   //--- Prepare data structures
   vector price_vec;
   price_vec.Init(prices);
   
   //--- Execute on GPU
   // Implementation depends on specific kernel...
   
   return true;
}

const string MA_KERNEL = R"(
   __kernel void moving_average(
      __global const double* prices,
      __global double* ma,
      int length,
      int period)
   {
      int idx = get_global_id(0);
      
      if(idx >= period - 1 && idx < length)
      {
         double sum = 0.0;
         for(int i = 0; i < period; i++)
         {
            sum += prices[idx - i];
         }
         ma[idx] = sum / period;
      }
      else if(idx < period - 1)
      {
         ma[idx] = 0.0;  // Not enough data
      }
   }
)";