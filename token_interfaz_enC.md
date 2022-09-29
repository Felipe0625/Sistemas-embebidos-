# Sistemas-embebidos- Daniel Felipe Sanchez- Sergio Molina 
#include <windows.h>
#include <sys/time.h>
#include <stdio.h>
#include <conio.h>
#define    BUFFERLENGTH 256
int main(int argc, char *argv[])
{
	HANDLE hComm;
	DWORD MORO;
	BOOL Write_Status;
	DCB dcbSerialParams;				
	COMMTIMEOUTS timeouts = { 0 };
	BOOL  Read_Status;                      
	DWORD dwEventMask;						
	char  *SerialBuffer[BUFFERLENGTH+1];              
	DWORD NoBytesRead;                    
	int i = 0,sincro=0;
	unsigned long clave=0x8F5D6A9B;
	char text[10];
	struct timeval stop, start;
	unsigned long tokenLocal =0;
	unsigned long tiempo;
	time_t hi;
	struct tm *info;
	char horaC[10];
	char claved[8];
	unsigned char a[4];
		
	

	hComm = CreateFileA("\\\\.\\COM5",
		GENERIC_READ | GENERIC_WRITE,
		0,    
		NULL,
		OPEN_EXISTING, 
		0,    
		NULL  
	);

	if (hComm == INVALID_HANDLE_VALUE)
	{
		
		if (GetLastError() == ERROR_FILE_NOT_FOUND)
		{
			puts("no se puede abrir el puerto");
			return 0;
		}

		//puts("invalid handle value!");
		return 0;
	}
	else
	 // printf("opening serial port successful");

	dcbSerialParams.DCBlength = sizeof(dcbSerialParams);

	Write_Status = GetCommState(hComm, &dcbSerialParams);     //retreives  the current settings

	if (Write_Status == FALSE) {
		printf("\n   Error! in GetCommState()");
		CloseHandle(hComm);
		return 1;
	}


	dcbSerialParams.BaudRate = CBR_57600;      // Setting BaudRate = 9600
	dcbSerialParams.ByteSize = 8;             // Setting ByteSize = 8
											  //dcbSerialParams.
	dcbSerialParams.StopBits = ONESTOPBIT;    // Setting StopBits = 1
	dcbSerialParams.Parity = ODDPARITY;      // Setting Parity = None

	Write_Status = SetCommState(hComm, &dcbSerialParams);  //Configuring the port according to settings in DCB




	
	timeouts.ReadIntervalTimeout = 50;
	timeouts.ReadTotalTimeoutConstant = 50;
	timeouts.ReadTotalTimeoutMultiplier = 10;
	timeouts.WriteTotalTimeoutConstant = 50;
	timeouts.WriteTotalTimeoutMultiplier = 10;
	if (SetCommTimeouts(hComm, &timeouts) == 0)
	{
		printf("Error setting timeouts\n");
		CloseHandle(hComm);
		return 1;
	}
		printf("ingrese su token");
		scanf("%X",&clave);
		for(char j=0;j<4;j++)
		{
         a[j]=(clave>>(24-(j*8))&0X000000FF);		
		}
			DWORD  NumWritten=256;
	DWORD  dNoOFBytestoWrite;              
	DWORD  dNoOfBytesWritten = 0;        

    dNoOFBytestoWrite = sizeof(a); 
     
    WriteFile(hComm, a,dNoOFBytestoWrite,&NumWritten, NULL);


	/*----------------------------- esperando para sincronizar----------------------------------------*/
	sincro=0;
	while(sincro==0)
	{
	printf("pulse 1 para sincronizar");
	scanf("%i",&sincro);
		if(sincro==1)
		{
			a[0]='z';
			gettimeofday(&start, NULL);
    hi = start.tv_sec;
    info = localtime(&hi);
    strftime(horaC, sizeof(horaC), "%H%M%S", info);

         sincro=1;
		}
		else
		{
		sincro=0;	
		}
	}
      
    dNoOFBytestoWrite = sizeof(a); 
     
    WriteFile(hComm, a,dNoOFBytestoWrite,&NumWritten, NULL);
		


	
	CloseHandle(hComm);
	printf("\n ==========================================\n");
	return 0;
}
