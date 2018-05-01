#include <stdio.h>
#define N 5
#define M 5

void Print_matrix(double C[], int n, int m) {
   int i, j;

   for (i = 0; i < n; i++) {
      for (j = 0; j < m; j++)
         printf("%f ", C[i+j*n]);
      printf("\n");
   }
}  /* Print_matrix */

__global__ void suma(double *a, double *b)
{
        int i=(threadIdx.x+blockIdx.x * blockDim.x)+1;
        int j=(threadIdx.y+blockIdx.y * blockDim.y)+1;
        if ((i<=N-2) && (j<=M-2)) {
                b[(i-1)+(j-1)*(N-2)]=(a[(i-1)+j*N]+a[(i+1)+j*N]+a[i+(j-1)*N]+a[i+(j+1)*N]+a[i+j*N])/5.0;
        }
}

int main()
{
        double a[N*M], b[(N-2)*(M-2)];
        int i,j;
        double *dev_a, *dev_b;
                //reservar memoria en GPU
        cudaMalloc((void **) &dev_a, N*M*sizeof(double) );
        cudaMalloc((void **) &dev_b, (N-2)*(M-2)*sizeof(double) );

        //rellenar vectores en CPU
        for (i=0;i<N;i++)
        {
                for (j=0;j<M;j++){
                        a[i+j*N]=i+j;
                }

        }
        //Print_matrix(a,N,M);

        //enviar vectores a GPU
        cudaMemcpy( dev_a, a, N*M*sizeof(double) , cudaMemcpyHostToDevice );
        cudaMemcpy( dev_b, b, (N-2)*(M-2)*sizeof(double) , cudaMemcpyHostToDevice );


        dim3 thr_p_block(N-2,M-2);

        //llamar al Kernel
        suma<<<1,thr_p_block>>>(dev_a,dev_b);
        //obtener el resultado de vuelta en la CPU
        cudaMemcpy( b, dev_b, (N-2)*(M-2)*sizeof(double), cudaMemcpyDeviceToHost );
        //imprimir resultado
        printf("__________________________A\n");
        Print_matrix(a,N,M);
        printf("__________________________B\n");
        Print_matrix(b,N-2,M-2);

        //liberar memoria en la GPU
        cudaFree(dev_a) ;
        cudaFree(dev_b) ;

        return 0;
}
