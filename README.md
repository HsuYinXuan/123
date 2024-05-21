#include<stdio.h>
#include<stdlib.h>
#include<string.h>


typedef struct{
	char program_name[100];
	int address;
	int length;
}Program;

void objInit(Program program[]){
	for (int i=0;i<10;i++){
		strcpy(program[i].program_name," ") ;
		program[i].address = 0;
		program[i].length = 0;
	}
}

int main(int argc, char *argv[]){
    if (argc < 2) {
        puts("請輸入PROGADDR和至少一個檔案");
    }
    
	Program program[10];//創10個物件 
	objInit(program);//初始化 

    long int PROGADDR = strtol(argv[1], NULL, 16);//把progaddr放進去 
    int CSADDR=PROGADDR;//第一個cs的addr就是progaddr 阿CSADDR 就是該CS開始位置
	 
    int CSLTH;//cs的長度(H......後面的數字) 
    //printf("%04X",CSADDR);
    int t = 2;//從第二個檔案開始 因為0是a 1是progaddr 
	int e_flag=0;
	
    while (t < argc) {//重複每個cs 
        FILE* f = fopen(argv[t], "r");
        if (f == NULL) {
            puts("檔案打不開 慘囉");
            return 1;
        }
        program[1].address=PROGADDR;
		e_flag=0;
        char buf[500];
        memset(buf, 0, sizeof(buf));
        while (fgets(buf, sizeof(buf), f) != NULL) {
            if (buf[0] == 'H') {
                int i = 0;
                int flag=0;//0:csname沒重複 1:csname有重複 
                
                //------------------------放program名稱 -----------
                while (buf[i] != ' ' && i < 6) {
                	program[t].program_name[i]=buf[i+1];
                    i++;
                }
                program[t].program_name[i] = '\0';
                //-------------------------------------------------
                
				if(t>2){
					for(int j=2;j<t;j++){
						if(strcmp(program[t].program_name ,program[j].program_name)==0){
							flag+=1;
						}
					}
				}
				if(flag>=1){//有重複的csname 
					printf("ERROR\n");
					e_flag=1;
					break;
				}else{
                	CSLTH = strtol(buf+7, NULL, 16);
                	program[t].length = strtol(buf+7, NULL, 16);
                	//CSADDR += program[t-1].length;
                	program[t].address = program[t-1].address+program[t-1].length;
                	printf("%6s \t \t %04lX \t %04lX\n", program[t].program_name,program[t].address, program[t].length);
				}
				
            } else if (buf[0] == 'D') {
                char tmp[7];
                memset(tmp, 0, sizeof(tmp));
                
                int end = 0;
                for (int i = 0; buf[i] != '\0'; i++) {
                    end = i;
                }
                buf[end] = '\0';

                for (int i = 1; i < 500; i += 12) {
                    long int len = program[t].address;
                    if (buf[i] == '\0') break;
                    char lablename[7];
                    memset(lablename, 0, sizeof(lablename));
                    int j = 0;
                    while (buf[i+j] != ' ' && j < 6) {
                        lablename[j] = buf[i+j];
                        j++;
                    }
                    lablename[j] = '\0';
                    j = 6;
                    while (buf[i+j] != ' ' && j < 12) {
                        tmp[j-6] = buf[i+j];
                        j++;
                    }
                    tmp[j-6] = '\0';
                    len += strtol(tmp, NULL , 16);
                    printf("\t %-6s  %04lX\n", lablename, len);
                }
            }
        }
        
        memset(buf, 0, sizeof(buf)); 
        printf("\n");
        if(e_flag==1){
        	printf("有重複的cs name哦 錯誤錯誤錯誤錯誤\n");
        	break;
		}else{
			t++;
		}
		
    }
    return 0;
}

