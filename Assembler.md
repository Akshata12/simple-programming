simple-programming
==================
#include<stdio.h>
#include<conio.h>
#include<string.h>
#include<math.h>
#include<ctype.h>

typedef struct MOT
{
	char mnemonic[10];
	int machine_code;
	char Class[3];
	int length;
}MOT;

typedef struct IC
{
	int LC;
	char Class[3];
	int code;
	int reg;
	char op;
	int op2;
}IC;


typedef struct SYMTAB
{
	char symname[10];
	int address;
}SYMTAB;

typedef struct LITTAB
{
	char symname[10];
	int address;
}LITTAB;

typedef struct POOLTAB
{
	int pool;
}POOLTAB;

MOT MOTTAB[18]={{"STOP",0,"IS",1},
		{"ADD",1,"IS",2},
		{"SUB",2,"IS",2},
		{"MULT",3,"IS",2},
		{"MOVER",4,"IS",2},
		{"MOVEM",5,"IS",2},
		{"COMP",6,"IS",2},
		{"BC",7,"IS",2},
		{"DIV",8,"IS",2},
		{"READ",9,"IS",2},
		{"PRINT",10,"IS",2},
		{"DS",1,"DL",1},
		{"DC",2,"DL",0},
		{"START",1,"AD",0},
		{"END",2,"AD",0},
		{"LTORG",3,"AD",0},
		{"ORIGIN",4,"AD",0},
		{"EQU",5,"AD",0}};

IC ic[50];
int icp=0,ltp=1,stp=1,ptp=1;
int LC;
LITTAB littab[10];
POOLTAB pooltab[5];
SYMTAB symtab[10];
char field1[20],field2[20],field3[20],field4[20];
FILE *fp1;

void print_source()
{
	FILE *fp;
	char buffer[80];

	fp=fopen("first.txt","r");
	if(fp==NULL)
	   printf("\n\nFile not found !!");
	else
	 {
		while(fgets(buffer,80,fp))
		 {
		  // fscanf(buffer,"%s%s%s%s%s",field1,field2,field3,field4,field5);
		   printf("\n%s",buffer);
		 }
	 }
}

int search_MOTTAB(char* field1)
{


	int i;

	for(i=0;i<19;i++)
	{
	 if(strcmp(MOTTAB[i].mnemonic,field1)==0)
	 {
		return i;
	 }
	}
	return -1;
}

void passone()
{
	FILE *fp;
	int n,r,i,j,n1,n2;
	int k,l=0,x,y,add;
	char buffer[80],ch,temp[10];
	int ltorg_key=0;
	int ds_key=0;

	pooltab[ptp].pool=ltp;
	printf("\nLC\t(Class,Code)\tReg\tOp2");
	fp=fopen("first.txt","r");
	if(fp==NULL)
	  {
	    printf("Error in opening the file !!");
	    getch();
	    exit(0);
	  }

	   while(fgets(buffer,80,fp))
	   {
	   n=sscanf(buffer,"%s%s%s%s",field1,field2,field3,field4);
	   switch(n)
	   {
		case 1:
			r=search_MOTTAB(field1);
			if(strcmp(MOTTAB[r].mnemonic,"STOP")==0)
			{
			  ic[icp].LC=LC;
			  strcpy(ic[icp].Class,MOTTAB[r].Class);
			  ic[icp].code=MOTTAB[r].machine_code;
			  printf("\n%d\t(%s,%d)\t\t-\t-",ic[icp].LC,ic[icp].Class,ic[icp].code);
			  LC++;
			  icp++;
			}

			else if(strcmp(MOTTAB[r].mnemonic,"LTORG")==0)
			{
			   ic[icp].LC=LC;
			   strcpy(ic[icp].Class,MOTTAB[r].Class);
			   ic[icp].code=MOTTAB[r].machine_code;
			   printf("\n%d\t(%s,%d)\t\t-\t-",ic[icp].LC,ic[icp].Class,ic[icp].code);
			   ltorg_key=1;
			   LITTABFn(field1,ltorg_key);
			   ptp++;
			   pooltab[ptp].pool=ltp;
			   icp--;
			}
			else
			{
			   ic[icp].LC=LC;
			   strcpy(ic[icp].Class,MOTTAB[r].Class);
			   ic[icp].code=MOTTAB[r].machine_code;
			   printf("\n%d\t(%s,%d)\t\t-\t-",ic[icp].LC,ic[icp].Class,ic[icp].code);
			   ltorg_key=1;
			   LITTABFn(field1,ltorg_key);
			}
			//icp++;
			ltorg_key=0;
			break;

		case 2:
			r=search_MOTTAB(field1);
			if(strcmp(MOTTAB[r].mnemonic,"START")==0)
			{
			   ic[icp].LC=atoi(field2);
			   LC=ic[icp].LC;
			   //icp++;
			}
			else
			{
			   if(isalpha(field2[0]))
			   {
				for(i=0;i<strlen(field2);i++)
				{
					if(field2[i]=='+'||field2[i]=='-')
						break;
				}
				n1=i;
				for(i=0;i<n1;i++)
					temp[i]=field2[i];
				temp[i]='\0';
				k=SYMTABFn(temp,0);
				add=symtab[k].address;
				for(j=n1+1;j<strlen(field2);j++)
				{
					temp[l]=field2[j];
					l++;
				}
				n2=atoi(temp);
				if(field2[n1]=='+')
					LC=add+n2;
				else
					LC=add-n2;
				ic[icp].LC=LC;
				}

				else
				{
				LC=atoi(field2);
				ic[icp].LC=LC;
				}

			}
			//icp++;
			break;
		case 3:
			r=search_MOTTAB(field1);
			if(r==-1)
			{
			   r=search_MOTTAB(field2);
			   ic[icp].LC=LC;
			   strcpy(ic[icp].Class,MOTTAB[r].Class);
			   ic[icp].code=MOTTAB[r].machine_code;
			   ic[icp].reg=00;
			   ic[icp].op2=atoi(field3);
			   if(strcmp(field2,"DS")==0)
				 {

				ds_key=1;
				SYMTABFn(field1,ds_key);
				ic[icp].op='C';
				LC=LC+atoi(field3);
				printf("\n%d\t(%s,%d)\t\t%d\t(%c,%d)",ic[icp].LC,ic[icp].Class,ic[icp].code,ic[icp].reg,ic[icp].op,ic[icp].op2);
				 }
			   if(strcmp(field2,"DC")==0)
				 {
				ds_key=1;
				SYMTABFn(field1,ds_key);
				ic[icp].op='C';
				j=0;
				for(i=1;i<strlen(field3);i++)
				{
					temp[j]=field3[i];
					if(field3[i]=='\'')
					{
						temp[j]='\0';
						break;
					}
					j++;
				}
				//temp[j]='\0';
				ic[icp].op2=atoi(temp);
				//ic[icp].op2=atoi(field3);
				printf("\n%d\t(%s,%d)\t\t%d\t(%c,%d)",ic[icp].LC,ic[icp].Class,ic[icp].code,ic[icp].reg,ic[icp].op,ic[icp].op2);
				LC++;

				 }
			   if(strcmp(field2,"EQU")==0)
				 {
				  x=SYMTABFn(field3);
				  y=SYMTABFn(field1);
				  symtab[y].address=symtab[x].address;
				  printf("\n%d\t(%s,%d)\t\t-\t-",ic[icp].LC,ic[icp].Class,ic[icp].code);
				  LC++;
				 }

			   ds_key=0;
			}
			else
			{
			ic[icp].LC=LC;
			strcpy(ic[icp].Class,MOTTAB[r].Class);
			ic[icp].code=MOTTAB[r].machine_code;
			ic[icp].reg=reg(field2);
			if(field3[0]=='=')
			   {
				  ic[icp].op2=LITTABFn(field3);
				  ic[icp].op='L';
				  printf("\n%d\t(%s,%d)\t\t%d\t(%c,%d)",ic[icp].LC,ic[icp].Class,ic[icp].code,ic[icp].reg,ic[icp].op,ic[icp].op2);
				  LC=LC+2;
			   }
			else
			   {
				ic[icp].op2=SYMTABFn(field3);
				ic[icp].op='S';
				printf("\n%d\t(%s,%d)\t\t%d\t(%c,%d)",ic[icp].LC,ic[icp].Class,ic[icp].code,ic[icp].reg,ic[icp].op,ic[icp].op2);
				LC=LC+2;
			   }
		    }
			icp++;
			break;
		case 4:
			x=SYMTABFn(field1);
			ic[icp].op2=x;
			symtab[x].address=LC;
			r=search_MOTTAB(field2);
			ic[icp].LC=LC;
			strcpy(ic[icp].Class,MOTTAB[r].Class);
			ic[icp].code=MOTTAB[r].machine_code;
			ic[icp].reg=reg(field3);

			if(field4[0]=='=')
			   {
				  ic[icp].op2=LITTABFn(field4);
				  ic[icp].op='L';
				  printf("\n%d\t(%s,%d)\t\t%d\t(%c,%d)",ic[icp].LC,ic[icp].Class,ic[icp].code,ic[icp].reg,ic[icp].op,ic[icp].op2);
				  LC=LC+2;
			   }
			else
			   {
				ic[icp].op2=SYMTABFn(field4);
				ic[icp].op='S';
				printf("\n%d\t(%s,%d)\t\t%d\t(%c,%d)",ic[icp].LC,ic[icp].Class,ic[icp].code,ic[icp].reg,ic[icp].op,ic[icp].op2);
				LC=LC+2;
			   }
			icp++;
			break;
	   }
	}
//}

//void print()
//{
	//int i;
	ch=getch();
	if(ch==13)
	{
	   clrscr();
	   printf("\n\nSYMBOL TABLE ");
	   printf("\n\nIndex\tSymbol\tAddress");
	   for(i=1;i<stp;i++)
		{
		printf("\n\n%d\t%s\t%d",i,symtab[i].symname,symtab[i].address);
		}
	   printf("\n\n\nLITERAL TABLE");
	   printf("\n\nIndex\tLiteral\tAddress");
	   for(i=1;i<ltp;i++)
		{
		printf("\n%d\t%s\t%d",i,littab[i].symname,littab[i].address);
		}
	   printf("\n\n\nPOOLTAB TABLE");
	   printf("\n\nIndex");
	   for(i=1;i<=ptp;i++)
		{
		printf("\n\n#%d",pooltab[i].pool);
		}
	}
}

void icfile()
{
	 FILE *fp1;
	 fp1=fopen("ic1.txt","w");
	 if(fp1==NULL)
		printf("\n Error in opening file:");
	 else
	 {
		fwrite(ic, sizeof(struct IC),50,fp1);
	 }
	 fclose(fp1);

}
void passtwo()
{
	char buffer[50];
	int i,add;
	printf("\nLC\t(Class,Code)\tReg\tOp2");
	for(i=0;i<icp;i++)
	   {
		if(ic[i].op=='L')
		{
			add=littab[ic[i].op2].address;
			printf("\n+\t%d\t\t%d\t%d",ic[i].code,ic[i].reg,add);
		}
		else if(ic[i].op=='S')
		{
			add=symtab[ic[i].op2].address;
			printf("\n+\t%d\t\t%d\t%d",ic[i].code,ic[i].reg,add);
		}
		else if(ic[i].op=='C')
		{
			add=ic[i].op2;
			printf("\n+\t%d\t\t%d\t%d",ic[i].code,ic[i].reg,add);
		}
		else if(ic[i].op2=='\0')
		{
			printf("\n+\t%d\t\t-\t-",ic[i].code);
		}
		/*else
		{
			printf("\n+\t%d\t\t%d\t(%c,%d)",ic[i].code,ic[i].reg,ic[i].op,ic[i].op2);
		}*/
	   }
}

int LITTABFn(char* field3,int ltorg_key)
{
  int i,flag=0;
  if(ltorg_key==1)
  {
	i=1;
	for(i=pooltab[ptp].pool;i<ltp;i++)
	{
		littab[i].address=LC;
		LC++;
		icp++;
	}

  }

  else
  {

  for(i=pooltab[ptp].pool;i<ltp;i++)
	{
		if(strcmp(littab[i].symname,field3)==0)
		{
			flag=1;
			return i;
		}
	}
	if(flag!=1)
	{
		strcpy(littab[ltp].symname,field3);
		ltp++;
	}
  }
  return ltp-1;
}

int SYMTABFn(char* field3,int ds_key)
{
  int i,flag=0;
  if(ds_key==1)
  {
	for(i=1;i<stp;i++)
	{
		if(strcmp(symtab[i].symname,field3)==0)
		{
			symtab[i].address=LC;
		}
	    /*	else
		{
		 strcpy(symtab[stp].symname,field3);
		 stp++;
		 return stp-1;
		}*/

	}
  }
  else
  {
  for(i=1;i<=stp;i++)
	if(strcmp(symtab[i].symname,field3)==0)
		{
		flag=1;
		return i;
		}
  if(flag!=1){
  strcpy(symtab[stp].symname,field3);
  stp++;
  return stp-1;
 }
}
return -1;
}

int reg(char field2[5])
{
	if(strcmp(field2,"AREG")==0)
	  return 1;
	if(strcmp(field2,"BREG")==0)
	  return 2;
	if(strcmp(field2,"CREG")==0)
	  return 3;
	if(strcmp(field2,"DREG")==0)
	  return 4;
 return 0;
}

void main()
{
	int i;
	char ch;
	clrscr();
	print_source();
	ch=getch();
	if(ch==13)
	{
		clrscr();
		passone();
	}
	ch=getch();
	if(ch==13)
	{
		clrscr();
		icfile();
	}
	ch=getch();
	if(ch==13)
	{
		clrscr();
		passtwo();
	}
	getch();
}
