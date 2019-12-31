---
title: 리눅스 모듈 프로그래밍 1 - 모듈 빌드&기본 예제
categories: kernel, linux
comments: true
---
## 명령어

- lsmod - 현재 로드된 모듈 확인
- rmmod [모듈] - 모듈 언로드
- insmod [모듈] - 모듈 로드
- modprobe - insmod에서 의존성 기능 추가한거

- /var/log/messages에서 로그 확인

## Basic code - hello world

```c
#include <linux/module.h> //모든 리눅스 모듈에 필요
#include <linux/kernel.h> //printk 매크로

int init_module(void) {
  printk(KERN_INFO "hello kernel!\n"); // 로그 레벨.
  return 0 //0이 아니면 로딩 안됨.
}

void cleanup_module(void) {
  printk(KERN_INFO "bye kernel!\n");
}
```

- init_module(void) -> 모듈이 로드될때
- cleanup_module(void) -> 모듈이 언로드될때.

실제로 insmod와 rmmod로 로드, 언로드 해보면 메시지가 뜨는것을 확인할수있음.

init_module과 cleanup_module의 역할을 하는 함수를 다른 이름으로 만들수있음. module_init, module_exit 매크로를 이용하면 된다.

```c
#include <linux/module.h> 
#include <linux/kernel.h>

static int __init hello_init(void) {
  printk(KERN_INFO "hello kernel!\n"); 
  return 0;
}

static void __exit hello_exit(void) {
  printk(KERN_INFO "bye kernel!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

- 여기서 \_\_init,\_\_initdata,\_\_exit,\_\_exitdata 매크로는 built-in driver에서만 동작하는 매크로인데, 커널 초기화과정 이후에 해제되는 영역에 데이터를 배치해서 메모리를 절약할수있음.
  - 단, built-in이 아닌 모듈로 적재되는 드라이버에 대해서는 아무 효과도 없다.
  - https://kkoroke.tistory.com/entry/init-initdata-exit-exitdata

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
static int hello3_data __initdata = 3;
static int __init hello_3_init(void)
{
	printk(KERN_INFO "Hello, world %d\n", hello3_data);
	return 0; 
}
static void __exit hello_3_exit(void) 
{
printk(KERN_INFO "Goodbye, world 3\n"); 
}
module_init(hello_3_init);
module_exit(hello_3_exit);
```

- 위와같이 \_\_initdata 매크로를 이용해서 초기화 함수를 위한 변수를 만들어줄수있다.

### Compile

- linux/documentation/kbuild/modules.txt에서 자세하게 나와있음.

```
obj-m += helloworld.o
KERNEL_SOURCE := [커널소스경로]
PWD := $(shell pwd)
all:
	$(MAKE) -C $(KERNEL_SOURCE) SUBDIRS=${PWD} modules
clean
	$(MAKE) -C $(KERNEL_SOURCE) SUBDIRS=${PWD} clean
```

- 이렇게 빌드하면 .ko 확장자의 모듈이 생김. 
- __그리고 기본적으로 init_module, cleanup_module외에는 static 키워드를 꼭 붙어야한다.__ - 커널 변수들끼리의 이름 충돌때문에.

## 인자받는 모듈 만들기.

- 커맨드라인 변수들을 전역변수로 선언하고 module_parm 매크로를 이용하자.
  - module_param은 3가지 인자를 입력받음. 변수이름, 타입, 권한
    - 권한은 sysfs에 대응하는 파일을 위해 존재한다는데, 이건 잘 모르므로 나중에 추가해야겠따.
  - 배열은 module_parm_array라는 매크로를 이용.
    - 세번째 인자에 count variable에 대한 포인터를 넘겨주고 4번째 인자에 권한을 넣는다.
    - count를 무시하고 NULL을 대신 넘겨줄수도 있음.
    - argc같은 느낌.
  - MODULE_PARM_DESC는 변수들에 대한 설명을 추가시켜주는 매크로.
    - modinfo로 모듈의 정보를 확인할때 보임.

```c
#include <linux/module.h>
#include <linux/moduleparam.h>
#include <linux/kernel.h>
#include <linux/init.h>

static int integer = 1;
static char *string = "asdf";
static int integerArr[2] = {0,0};

module_param(integer,int, S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP);
MODULE_PARM_DESC(integer, "integer.");
module_param(string, charp, 0000);
MODULE_PARM_DESC(string, "string.");
module_param_array(integerArr, int, NULL, 0000);
MODULE_PARM_DESC(integerArr, "integerArr.");

static int __init param_test_init(void)
{
	int i;
	printk(KERN_INFO "integer: %d\n", integer);
	printk(KERN_INFO "string: %s\n", string);
	for(i = 0; i < 2; i++) {
		printk(KERN_INFO "integerArr[%d] = %d\n", i, integerArr[i]);
	}
	return 0;
}
static void __exit param_test_exit(void)
{
	printk(KERN_INFO "bye kernel!\n");
}
module_init(param_test_init);
module_init(param_test_exit);
```


