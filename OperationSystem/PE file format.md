# 파일 포맷

## 정의
- ( = 파일 형식 )
- 저장을 목적으로 정보를 컴퓨터 파일로 `인코딩`하는 특별한 `방식`

# PE 파일 포맷

## 정의
- Windows 운영체제에서 사용되는 파일 형식 중 하나
- PE 포맷을 사용하는 파일 확장자 : 
    1) `실행 파일` 계열       Ex) EXE, SCR
    2) 라이브러리 계열      Ex) DLL, OCX
    3) 드라이버 계열        Ex) SYS
    4) 오브젝트 파일 계열   Ex) OBJ

## PE 파일 구조
PE 파일은 크게 PE Header와 PE Body로 나눌 수 있음
PE Header는 파일이 실행되기 위해 필요한 모든 정보들이 `구조체 형식`으로 저장

=> PE File Format은 PE header 구조체를 파악하면 파악완료!

아래는 PE 파일의 구조입니다

||
| :------------: |
| IMAGE_DOS_HEADER| 
| DOS Stub Program|
| Section header|
|NULL|
| Section 1|
|NULL|
| Section 2|
|NULL|
| Section 3|
|NULL|
|...|
|NULL|
| Section N|
|NULL|

- 이 구조는 프로그램이 메모리에 적재될 때에도 순서를 거의 유지함
- 단, Section의 크기, 위치등 모양은 달라짐
- PE Header : DOS header ~ Section header
- PE Body : Section1 ~ Section N
- 위치 표현 : 파일 - offset , 메모리 - VA(Virtual Address)

### VA & RVA
VA(Virtual Address) : 프로세스 가상 메모리의 절대 주소
RVA(Relative Virtual Address) : 기준 위치 에서부터의 상대 주소

    RVA + Image = VA

### 1.DOS Header (IMAGE_DOS_HEADER)

    typedef struct _IMAGE_DOS_HEADER {
        WORD    e_magic;       // DOS signature : 4D5A ("MZ")
        WORD    e_cblp;
        WORD    e_cp;
        WORD    e_crlc;
        WORD    e_cparhdr;
        WORD    e_minalloc;
        WORD    e_maxalloc;
        WORD    e_ss;
        WORD    e_sp;
        WORD    e_csum;
        WORD    e_ip;
        WORD    e_cs;
        WORD    e_lfarlc;
        WORD    e_ovno;
        WORD    e_res[4];
        WORD    e_oemid;
        WORD    e_oeminfo;
        WORD    e_res2[10];
        LONG    e_lfanew;       // offset to NT header
    } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER

- e_magic : DOS signature (4D5A => ASCII 값 "MZ)
- e_Ifanew : NT header의 옵셋을 표시(가변적)

- 모든 PE 파일의 시작부분(e_magic)에는 DOS signature("MZ") 존재
- e_Ifanew 값이 가리키는 위치에는 NT header 구조체(IMAGE_NT_HEADERS) 존재 

### 2.DOS Stub
DOS Header 밑에 존재, 존재여부는 옵션, 크기는 가변적
- DOS Stub 가 없어도 파일 실행에는 문제가 없음


### 3.NT Header (IMAGE_NT_HEADERS)

    typedef struct _IMAGE_NT_HEADERS {
        DWORD Signature;        //PE Signature : 50450000("PE"00)
        IMAGE_FILE_HEADER FileHeader;
        IMAGE_OPTIONAL_HEADER32 OptionalHeader;
    } IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS;

- 위의 코드는 32bit 용, 64bit는 세번째 멤버가 IMAGE_OPTIONAL_HEADEER64
- 첫번쨰 멤버 : Signature("PE"00) (변경불가)
- FileHeader, OptionalHeader 구조체 멤버있음


#### 3-1. IMAGE_FILE_HEADER

    typedef struct _IMAGE_FILE_HEADER {
        WORD    Machine;
        WORD    MunberOfSections;
        DWORD   TimeDateStamp;
        DWORD   PointerToSymbolTable;
        WORD    SizeOfOptionalHeader;
        WORD    Characteristics;
    } IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;

 - Machine : Machine 넘버는 CPU 별로 고유한 값 (32 bit intel 호환은 14Ch)
 - NumberOfSections : 섹션의 갯수 (반드시 0보다 커야 함) 
 - SizeOfOptionalHeader : IMAGE_OPTIONAL_HEADER32 구조체의 크기
 - Characteristics : 파일의 속성을 나타냄 (bit OR 형식으로 조합)
 - TimeDateStamp : 파일의 실행에 영향 X, 해당 파일의 빌드 시간 나타냄

#### 3-2. IMAGE_OPTIONAL_HEADER32

    typedef struct _IMAGE_DATA_DIRECTORY {
        DWORD VirtualAddress;
        DWORD Size;
    } IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;

    #difine IMAGE_NUMBEROF_DIRECTORY_ENTRIES    16

    typedef struct _IMAGE_OPTIONAL_HEADER {
        WORD    Magic;
        BYTE    MajorLinkerVersion;
        BYTE    MinorLinkerVersion;
        DWORD   SizeOfCode;
        DWORD   SizeOfInitializedData;
        DWORD   SizeOfUninitializedData;
        DWORD   AddressOfEntryPoint;
        DWORD   BaseOfCode;
        DWORD   BaseOfData;
        DWORD   ImageBase;
        DWORD   SectionAlignment;
        DWORD   FileAlignment;
        WORD    MajorOperatingSystemVersion;
        WORD    MinorOperatingSystemVersion;
        WORD    MajorImageVersion;
        WORD    MinorImageVersion;
        WORD    MajorSubsystemVersion;
        WORD    MinorSubsystemVersion;
        DWORD   Win32VersionValue;
        DWORD   SizeOfImage;
        DWORD   SizeOfHeaders;
        DWORD   CheckSum;;
        WORD    Subsystem;
        WORD    DllCharacteristics;
        DWORD   SizeOfStackReserve;
        DWORD   SizeOfStackCommit;
        DWORD   SizeOfHeapReserve;
        DWORD   SizeOfHeapCommit;
        DWORD   LoaderFlags;
        DWORD   NumberOfRvaAndSizes;
        IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
    } IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPRIONAL_HEADER32;

- Magic : 32bit = 10Bh, 64bit = 20Bh
- AddressOfEntryPoint : EP(Entry Point)의 RVA값을 가지고 있음
- ImageBase : PE 파일이 매핑되는 시작 주소를 나타냄
- SectionAlignment, FileAlignment : 각각 메모리, 파일에서 섹션의 최소단위를 나타냄
- SizeOfImage : PE Image가 차지하는 크기를 나타냄
- SizeOfHeader : PE header의 전체 크기를 나타냄
- Subsystem : 1) Driver file (*.sys)
              2) GUI 파일 (윈도우 기반 어플리케이션)
              3) CUI 파일 (콘솔 기반 어플리케이션)
- NumberOfRvaAndSizes : 마지막 멤버인 DataDirectory 배열의 갯수
- DataDirectory : IMAGE_DATA_DIRECTORY 구조체의 배열

### 4.Section Header
각 Section의 속성(property)을 정의한 것
왜 섹션으로 나누어서 저장할까? -> 프로그램의 안정성
1) code : 실행,읽기 권한
2) data : 비실행,읽기,쓰기 권한
3) resource : 비실행, 읽기 권한

    #define IMAGE_SIZEOF_SHORT_NAME

    typedef struct _IMAGE_SECTION_HEADER {
        BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];
        union {
                DWORD   PhysicalAddress;
                DWORD   VirtualSize;
        } Misc;
        DWORD   VirtualAddress;
        DWORD   SizeOfRawData;
        DWORD   PointerToRawData;
        DWORD   PointerToRelocations;
        DWORD   PointerToLinenumbers;
        WORD    NumberOfRelocations;
        WORD    NumberOfLinenumbers;
        DWORD   Characteristics;
    } IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;

- VirtualSize : 메모리에서 섹션이 차지하는 크기
- VirtualAdderss : 메모리에서 섹션의 시작 주소(RVA)
- SizeOfRawData : 파일에서 섹션이 차지하는 크기
- PointerToRawData : 파일에서 섹션의 시작 위치
- Characteristics : 섹션의 특징


# 프로그램 실행 구조

## 컴퓨터의 기본적인 구성요소
- CPU
- 메모리
- 하드디스크 : 실행파일은 하드디스크에 저장된다.

## 실행 과정

1) PE 포맷(헤더+바디)으로 구성된 실행 파일 클릭
2) 운영체제 내의 로더가 PE 헤더에 있는 정보를 분석
3) PE보디의 코드와 데이터 메모리에 적재
4) 이때, 메모리 구조 중 코드 영역과 데이터 영역에 자료가 들어감
5) 프로그램이 실행되면서 스택과 힙 영역에 데이터가 쌓임
6) 운영체제가 엔트리 포인트를 PE 헤더에서 찾아 그곳부터 프로그램 실행함

엔트리 포인트(Entry Point) = C 언어의 main함수
                          = PE파일 실행이 시작되는 주소


# CPU의 사용

- 여러 프로그램 동시 실행 시 번갈아 가며 CPU 사용
- CPU 사용이 끝나면 다른 프로그램에게 CPU 사용권한 넘김
- - 이때, 자신이 사용하고 있던 레지스터를 메모리 영역으로 복사 = 컨텍스트 스위칭