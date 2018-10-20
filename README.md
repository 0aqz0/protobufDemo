# protocal-buffers C++基础

## protobuf是什么

protobuf(Protocal Buffers)是google开发的一种用于序列化结构化数据（JSON、XML)的一种方式。你可以定义你的数据的结构，protobuf是语言中性的，可以使用C++、C#、GO、Java、Python来读写你的数据。

## 为什么使用protobuf

灵活、高效，只需要关于数据结构的描述.proto，就可以使用compiler自动生成进行编码、解析的class，这个class提供了getter和setter这些method读写protobuf。而且，protobuf有更强的拓展性（读取旧的格式）。

## 如何使用protobuf

- 在一个.proto文件定义消息类型

- 使用protobuf编译器编译

- 使用protobuf API发送和接受消息

### 1. 在一个.proto文件定义消息类型

新建一个.proto文件，里面定义你的数据结构（name、field），如下：

```protobuf
syntax = "proto2";

package tutorial;

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

.proto文件需要首先声明是哪一个package，以避免命名冲突。生成的class会被放在以package为名字的命名空间。

像Person、AddressBook这种就是name，像Person里面的name、id、email之类的就是field。field可以是一些基本的变量类型，例如bool、int32、float、double、string。不同message还可以嵌套，还可以定义enum类型。

每个field都有一个tag：

- required：必需的，没有初始化会报错
- optional：可选的，没有设置的话会使用默认值
- repeated：类似动态的数组

参考： [Protocol Buffer Language Guide](https://developers.google.com/protocol-buffers/docs/proto)。

### 2. 使用protobuf编译器编译

首先是下载protobuf，可以参考这个[链接](https://github.com/protocolbuffers/protobuf/tree/master/src)。

然后是在vs中新建一个项目protobufDemo：

![1539953568218](C:\Users\0AQZ0\AppData\Roaming\Typora\typora-user-images\1539953568218.png)

把编辑好的addressbook.proto消息定义文件放在工程文件夹下：

![1539953657218](C:\Users\0AQZ0\AppData\Roaming\Typora\typora-user-images\1539953657218.png)

同时将libprotobuf.dll、libprotoc.dll、protoc.exe放在工程文件夹下（这三个文件的原始路径：C:\Users\0AQZ0\myProject\vcpkg\packages\protobuf_x64-windows\tools\protobuf)：

![1539953742016](C:\Users\0AQZ0\AppData\Roaming\Typora\typora-user-images\1539953742016.png)

在当前目录下打开cmd，使用protobuf编译器进行生成class：

```bash
protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto
```

即

```bash
protoc -I=C:\Users\0AQZ0\Documents\ExerciseCode\C++\vs\protobufDemo\protobufDemo --cpp_out=C:\Users\0AQZ0\Documents\ExerciseCode\C++\vs\protobufDemo\protobufDemo C:\Users\0AQZ0\Documents\ExerciseCode\C++\vs\protobufDemo\protobufDemo\addressbook.proto
```

如下图：

![1539953908870](C:\Users\0AQZ0\AppData\Roaming\Typora\typora-user-images\1539953908870.png)

会看到生成一个头文件和一个源文件：

![1539953955786](C:\Users\0AQZ0\AppData\Roaming\Typora\typora-user-images\1539953955786.png)

然后在工程中加入这两个文件，就可以在程序中调用protobuf生成的class。

### 3. 使用protobuf API发送和接受消息

> accessors

每种message都会生成一个class，有很多成员函数（accessor）可以得到或者改变message中的信息：

set设置，has返回有没有，clear清空

对于字符串可以使用mutable，直接返回一个指向字符串的指针

对于repeated的field，可以检查size，根据index得到field的信息或者更新field的信息等

参考： [C++ generated code reference](https://developers.google.com/protocol-buffers/docs/reference/cpp-generated)。

> 一些通用的函数

标准的消息工具：

- `bool IsInitialized() const;`：检查是否所有的required的field都初始化了。
- `string DebugString() const;`：返回用于debug的消息。
- `void CopyFrom(const Person& from);`：复制别的消息。
- `void Clear();`：清空所有的元素到初始状态。

参考：[complete API documentation for `Message`](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.message.html#Message)。

序列化和解析的工具：

- `bool SerializeToString(string* output) const;`: 序列化消息并存储在给定的字符串里，格式是二进制的，字符串只是一个container.
- `bool ParseFromString(const string& data);`: 从字符串解析出消息.
- `bool SerializeToOstream(ostream* output) const;`: 把消息写到 C++ `ostream`.
- `bool ParseFromIstream(istream* input);`: 从 C++ `istream`解析消息.

> 发送消息和接收消息demo

```c++
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// This function fills in a Person message based on user input.
void PromptForAddress(tutorial::Person* person) {
	cout << "Enter person ID number: ";
	int id;
	cin >> id;
	person->set_id(id);
	cin.ignore(256, '\n');

	cout << "Enter name: ";
	getline(cin, *person->mutable_name());

	cout << "Enter email address (blank for none): ";
	string email;
	getline(cin, email);
	if (!email.empty()) {
		person->set_email(email);
	}

	while (true) {
		cout << "Enter a phone number (or leave blank to finish): ";
		string number;
		getline(cin, number);
		if (number.empty()) {
			break;
		}

		tutorial::Person::PhoneNumber* phone_number = person->add_phones();
		phone_number->set_number(number);

		cout << "Is this a mobile, home, or work phone? ";
		string type;
		getline(cin, type);
		if (type == "mobile") {
			phone_number->set_type(tutorial::Person::MOBILE);
		}
		else if (type == "home") {
			phone_number->set_type(tutorial::Person::HOME);
		}
		else if (type == "work") {
			phone_number->set_type(tutorial::Person::WORK);
		}
		else {
			cout << "Unknown phone type.  Using default." << endl;
		}
	}
}

// Iterates though all people in the AddressBook and prints info about them.
void ListPeople(const tutorial::AddressBook& address_book) {
	for (int i = 0; i < address_book.people_size(); i++) {
		const tutorial::Person& person = address_book.people(i);

		cout << "Person ID: " << person.id() << endl;
		cout << "  Name: " << person.name() << endl;
		if (person.has_email()) {
			cout << "  E-mail address: " << person.email() << endl;
		}

		for (int j = 0; j < person.phones_size(); j++) {
			const tutorial::Person::PhoneNumber& phone_number = person.phones(j);

			switch (phone_number.type()) {
			case tutorial::Person::MOBILE:
				cout << "  Mobile phone #: ";
				break;
			case tutorial::Person::HOME:
				cout << "  Home phone #: ";
				break;
			case tutorial::Person::WORK:
				cout << "  Work phone #: ";
				break;
			}
			cout << phone_number.number() << endl;
		}
	}
}

// Main function:  Reads the entire address book from a file,
//   adds one person based on user input, then writes it back out to the same
//   file.
int main(int argc, char* argv[]) {
	// Verify that the version of the library that we linked against is
	// compatible with the version of the headers we compiled against.
	GOOGLE_PROTOBUF_VERIFY_VERSION;

	tutorial::AddressBook address_book;

	// Add an address.
	PromptForAddress(address_book.add_people());

	fstream out("address_book.pb", ios::out | ios::binary | ios::trunc);
	if (!address_book.SerializeToOstream(&out)) {
		cerr << "Failed to write address book." << endl;
		return -1;
	}
	out.close();


	// Read the existing address book.
	fstream input("address_book.pb", ios::in | ios::binary);
	if (!input) {
		cout << "File not found.  Creating a new file." << endl;
	}
	else if (!address_book.ParseFromIstream(&input)) {
		cerr << "Failed to parse address book." << endl;
		return -1;
	}

	ListPeople(address_book);
	getchar();

	// Optional:  Delete all global objects allocated by libprotobuf.
	google::protobuf::ShutdownProtobufLibrary();

	return 0;
}
```

GOOGLE_PROTOBUF_VERIFY_VERSION
ShutdownProtobufLibrary

### 4. 向前兼容的一些规则

- you *must not* change the tag numbers of any existing fields.
- you *must not* add or delete any required fields.
- you *may* delete optional or repeated fields.
- you *may* add new optional or repeated fields but you must use fresh tag numbers (i.e. tag numbers that were never used in this protocol buffer, not even by deleted fields).

### reference:

[https://developers.google.com/protocol-buffers/docs/cpptutorial](https://developers.google.com/protocol-buffers/docs/cpptutorial)

[https://blog.csdn.net/mycwq/article/details/17606527](https://blog.csdn.net/mycwq/article/details/17606527)

[https://blog.csdn.net/k346k346/article/details/51754431](https://blog.csdn.net/k346k346/article/details/51754431)