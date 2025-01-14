## 입력 데이터를 플러그인 형식으로 추가

user id, product id 2개를 플러그인으로 변환

다음 장을 추가해서 09_03_more_plugin.md를 만들어서 추가 데이터들을 만드는 예제를 넣자.
command_option은 09_04_command_option으로 번호를 미루자.

플러그인: 고객번호, 제품번호, 고객국적, 언어, 고객유형:기업/개인/학생, 

각 플러그인마다 파일을 하나씩 만든다: country.rs lang.rs userid.rs product.rs serialver.rs
Json파일은 이중에서 골라서 시리얼번호를 만든다





요구사항은 언제나 바뀝니다. 시리얼 키 프로그램의 요구사항은 키 생성에 들어가는 입력 데이터의 종류나 크기일 것입니다.
그런데 입력 데이터가 더 많아지거나 달라질 수 있을것 같습니다. 고객이 많아지면 고객ID의 크기가 4글자에서 8글자가 될 수 있을 것입니다.
회사에 제품이 1개만 있는 것도 아니고, 제품 ID외에 제품의 종류에 대한 정보가 추가될 수도 있습니다. 단순히 제품의 ID만 입력할 게 아니라, 계약 만기 날짜나 고객의 국가 번호 등등 모든 제품마다 시리얼 키에 들어갈 입력 데이터가 다를 수도 있습니다.
그럼 제품마다 시리얼 키를 만드는 프로그램을 별도로 만들어야할까요? 그렇게 만든다고하면 사실상 동일한 코드가 여러번 사용될 것이 분명하기 때문에 효율이 좋은 방법은 아닐 것입니다.

결국 프로그램 사용자가 편리하게 프로그램을 사용하려면 여러가지 입력 데이터에 대한 처리 코드를 미리 만들어넣고, 필요에 따라 입력 데이터를 고를 수 있도록 옵션을 처리할 수 있게 만들어야합니다.
보통 이러한 디자인을 플러그인이나 드라이버 형태라고 부릅니다. 사용자 ID가 하나의 플러그인(혹은 드라이버)가 됩니다. 시리얼 키에 사용자 ID를 넣으려면 플러그인을 사용하면 됩니다.
시리얼 키에 사용자 ID가 안들어간다면 플러그인을 사용하지 않으면 됩니다. 플러그인이라는 말 그대로 필요에 따라 사용하기도하고 안쓰기도하면 됩니다.
우리같은 개발자는 다양한 플러그인을 준비해주면 됩니다. 그리고 최종 사용자(여기에서는 시리얼 키를 만드는 영업부서 직원이 될 수 있겠지요)가 필요한 플러그인을 골라서 사용하면 됩니다.

그럼 플러그인을 어떻게 만들수 있을까요? 가장 먼저 할 일은 우리가 사용할 각 입력 데이터의 공통적인 특성을 뽑아내는 것입니다.
지금 우리가 사용한 2개의 입력 데이터, 사용자 ID과 제품 ID의 특성들을 나열해보겠습니다.

사용자 ID
* 길이 4글자
* 숫자나 알파벳으로 이루어짐

제품 ID
* 길이 8글자
* 숫자나 알파벳으로 이루어짐

숫자나 알파벳으로 이루어졌다는 공통 특성이 나왔습니다. 그런데 길이가 다르네요. 근데 길이 다르다고 더 이상 공통 특성이 없는 것일까요? 아닙니다.
길이라는 특성을 가지고 있다는 것, 그리고 그 길이가 미리 정해져있다는 것 자체도 공통의 특성입니다.
예를 들어 네트워크로 입력받는 데이터같은 경우는 길이가 미리 정해져있지 않거나, 너무나 큰 데이터일 수 있습니다.
그런 데이터는 길이라는 특성이 없다고 볼 수도 있습니다. 하지만 우리가 사용할 데이터는 크기가 쉽게 다룰 수 있도록 작고 유한합니다.

그리고 각 입력 데이터에서 서로 다른 특성도 알 수 있습니다.
데이터의 길이가 다릅니다. 결국 두 데이터가 숫자와 알파벳으로 이루어져있고 길이가 있다는 공통 특징과, 길이가 다르다는 고유한 특징을 도출해내었습니다.
예를 들어 앞으로 각 입력 데이터를 위한 구조체를 만들 때 C++에서 말하는 템플릿같은 것으로 구조체를 만들 수 있습니다.
C에서는 매크로를 활용해서 구조체를 만들어낼 수도 있겠네요.

이제 데이터의 특성을 봤으니 그 다음 각 데이터를 처리하는 방법에는 어떤 공통적인 특성이 있는지 확인해보겠습니다.
일단 두 데이터를 입력받는 방법이 같습니다. 프로그램 사용자에게 입력을 달라는 메세지를 터미널에 출력하고, 터미널에서 입력을 받습니다.
입력을 받을 때, 각 입력 데이터의 특성들을 활용할 수 있으므로, 입력을 받는 코드 또한 공통적으로 사용할 수 있는 부분이 많을 것입니다.

그럼 두 입력 데이터를 위한 구조체부터 만들어보겠습니다.
각 데이터는 길이와 숫자/알파벳으로 이루어진 문자열이니 다음과 같은 구조체로 데이터를 표현할 수 있겠습니다.

```rust
pub struct InputData {
    pub name: String,
    pub digit: i32,
    pub id: Option<String>,
}

pub struct<T: InputData> UserID {
    pub data: T, // 공통 데이터는 InputData 구조체로 묶어서 관리
                         // 개별 데이터는 구조체의 필드로 선언
}

pub struct ProductID {
    pub data: InputData, // 공통 데이터는 InputData 구조체로 묶어서 관리
                         // 개별 데이터는 구조체의 필드로 선언
}
```

데이터의 형태가 동일하니 사용자에게서 두 데이터를 입력받거나, 구조체에 있는 데이터를 읽어오는 코드가 동일하게됩니다.
같은 코드에 단지 다른 데이터를 적용할 수 있게 됩니다. 서로 다른 구조체에 같은 코드가 있다면 바로 트레이트로 만들 수 있습니다. 
다음은 예제 코드를 보시면 공통 데이터를 얻어오는 트레이트 함수를 각각의 입력 데이터마다 별도로 구현하고, 공통 데이터를 받아온 이후부터 공통 데이터 자체를 처리하는 코드는 트레이트의 디폴트 구현으로 만든 예제입니다.

```rust
trait GenSerialData {
    fn get_input_from_user(&mut self) {
        let inputdata = self.return_input_data();
        println!(
            "Please input {}-digits for {}: ",
            inputdata.digit, inputdata.name
        );
        inputdata.id = Some(get_user_input());
    }

    fn get_data_from_struct(&mut self) -> Option<&str> {
        let inputdata = self.return_input_data();
        inputdata.id.as_ref().map(|x| x.as_str())
    }

    fn get_length(&mut self) -> usize {
        self.return_input_data().digit
    }

    fn return_input_data(&mut self) -> &mut InputData;
}
```

이전에 트레이트를 소개할 때는 트레이트 정의에 각 함수에 대한 구현은 없었습니다.
하지만 GenSerialData라는 트레이트는 몇가지 함수에 대한 코드가 이미 구현되어있습니다.
이렇게 트레이트 정의에 미리 구현되어있는 코드를 트레이트의 디폴트 구현이라고 부릅니다.
앞으로 GenSerialData 트레이트를 사용하는 구조체들은 get_input_from_user와 get_data_from_struct, get_lenth 함수들을 구현할 필요가 없습니다.
단지 디폴트 구현이 없는 return_input_data 함수만 구현하면 됩니다.

사용자 ID를 관리하는 UserID라는 구조체 타입을 만들고 GenSerialData 트레이트를 구현해보겠습니다.

```rust
use crate::{GenSerialData, InputData};

pub struct UserID {
    pub data: InputData, // 공통 데이터는 InputData 구조체로 묶어서 관리
                         // 개별 데이터는 구조체의 필드로 선언
}

impl UserID {
    pub fn new(digit: usize) -> Self {
        UserID {
            data: InputData {
                name: "UserID".to_owned(),
                digit,
                id: None,
            },
        }
    }
}

impl GenSerialData for UserID {
    fn return_input_data(&mut self) -> &mut InputData {
        &mut self.data
    }
}
```

UserID가 어떤 필드들을 갖는지는 상관없습니다.
저는 UserID 구조체 안에 InputData 구조체를 저장했지만, 사실 꼭 그렇게 할 필요는 없습니다.
UserID 구조체를 어떻게 구현하는 가는 GenSerialData와 아무 상관없습니다.
중요한 것은 GenSerialData를 구현할 때 return_input_data 함수가 InputData의 가변 참조를 반환하기만 하면 됩니다.
GenSerialData라는 트레이트와 InputData 구조체 타입이 UserID라는 개별 입력 데이터와 프로그램간의 인터페이스가 되는 것입니다.



```rust
use crate::{GenSerialData, InputData};

pub struct ProductID {
    pub data: InputData, // 공통 데이터는 InputData 구조체로 묶어서 관리
                         // 개별 데이터는 구조체의 필드로 선언
}

impl ProductID {
    pub fn new(digit: usize) -> Self {
        ProductID {
            data: InputData {
                name: "UserID".to_owned(),
                digit,
                id: None,
            },
        }
    }
}

impl GenSerialData for ProductID {
    fn return_input_data(&mut self) -> &mut InputData {
        &mut self.data
    }
}
```



```bash
g$ cargo run --bin serial_project_step3
   Compiling my-rust-book v0.1.0 (/home/gkim/study/quick-guide-rust-programming)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.51s
     Running `target/debug/serial_project_step3`
Please input 4-digits for UserID: 
1234
Please input 8-digits for UserID: 
qwerasdf
Plain serial: 1234qwerasdf
Encrypted serial: 3OvuVy1IXj5veDI61Mszjg==
Decrypted serial: 1234qwerasdf
Verify User ID: 1234
Verify Product ID: qwerasdf
```