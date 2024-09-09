# Color roles

Color role은 **UI 요소에 어떤 색상을 적용할지 결정**하기 위해, **색상에 역할을 부여**한 것이다. 

- Color role은 **머티리얼 컴포넌트에 매핑**된다. 
  - static baseline scheme 또는 dynamic color 중에 어떤 걸 사용하든, color role이 사용된다.
  - 커스텀 머티리얼 컴포넌트를 만드는 경우, color role에 따라 적절히 매핑해줘야 한다. 
- Color role은 **접근성을 보장**한다. 
  - 컬러 시스템은 접근성 있는 color 쌍을 기반으로 하며, 이는 접근성을 보장하기 위해 최소 3:1 대비를 제공한다. 
- Color role은 **토큰화** 되어있다. 
  - 디자인 토큰은 디자인 시스템의 시각적 스타일의 일부인, 재사용 가능한 작은 디자인 결정사항을 나타낸다.

<img width="800" src="https://github.com/user-attachments/assets/c2d22d5a-f0b0-4b39-bff5-63a2b76764b2"/>

## General Concepts 

* **Surface** : 화면의 배경이나 강조도가 낮은 넓은 영역에 사용
* **Primary, Secondary, Tertiary** : 포그라운드 컴포넌트를 강조하거나 강조하지 않는 데 사용되는 Accent color
* **Container** : 버튼과 같은 포그라운드 컴포넌트의 색상을 채우는 데 사용되는 컬러 (텍스트, 아이콘에 사용 X)
* **On-** : 쌍을 이루는 부모 색상 위에 텍스트, 아이콘 색상을 지정할 때 사용
* **-Variant** : 쌍을 이루는 non-variant 색상보다 덜 강조하고 싶을 때 사용

<img width="800" src="https://github.com/user-attachments/assets/27fa7d25-3238-4a46-bc28-87d27ac621b4"/>

## Primary 

FAB, 강조도가 높은 버튼, 활성 상태 등 **UI 전체에서 가장 눈에 띄는 컴포넌트**에 사용 

- **Primary** : 표면 대비 강조도가 높은 fill color 
- **On primary** : primary 위의 텍스트 및 아이콘
- **Primary container** : FAB과 같은 컴포넌트처럼 표면 대비 눈에 띄는 fill color 
- **On primary container** : primary container 위의 텍스트 및 아이콘

<img width="800" src="https://github.com/user-attachments/assets/1c9b25bd-e117-42d8-a846-92d815cd3eab">

## Secondary 

필터 칩과 같이 **UI에서 덜 눈에 띄는 컴포넌트**에 사용 

- **Secondary** : 표면에서 덜 눈에 띄는 fill color 
- **On Secondary** : secondary 위의 텍스트 및 아이콘 
- **Secondary Container** : tonal button 처럼 표면에서 덜 눈에 띄는 fill color 
- **On secondary container** : secondary container 위의 텍스트 및 아이콘 

<img width="800" src="https://github.com/user-attachments/assets/c6ba820b-89ac-4d6d-bdce-e228d13bb01e">

## Tertiary 

**primary, secondary 사이의 균형**을 맞추거나, 입력 필드와 같은 요소에 **주의를 집중시키는 대조적인 accent color**가 필요할 때 사용 

보다 폭넓은 색상 표현을 지원하기 위해, tertiary 색상은 **디자이너 재량에 따라 적용**할 수 있다. 

- **Tertiary** : 표면에 대한 보색 컬러  
- **On Tertiary** : tertiary 위의 텍스트 및 아이콘 
- **Tertiary container** : 입력 필드와 같은 표면에 대한 보색 컨테이너 컬러 
- **On tertiary container** : tertiary container 위의 텍스트 및 아이콘 

<img width="800" src="https://github.com/user-attachments/assets/894c779c-a100-403b-bc4c-9a55b8437ecd">

## Error

부정확한 비밀번호와 같은 에러 상태를 나타내기 위한 컬러 

- **Error** : 긴급함을 나타내는 주의를 끄는 색상 
- **On error** : error 위의 텍스트 및 아이콘 
- **Error container** : 표면 대비 주의를 끄는 fill color 
- **On error container** : error container 위의 텍스트 및 아이콘 

Error color는 정적 컬러의 예시이다. 모든 dynamic color scheme에서도 정적으로 만들어진다. 

## Surface 

보다 중성적인 **배경** 색상, 카드, 시트, 다이얼로그 같은 **컨테이너**의 색상으로 사용 

- **Surface** : 배경에 대한 디폴트 색상 
- **On surface** : surface 위의 텍스트 및 아이콘 
- **On surface variant** : on surface 보다 더 낮은 강조 

강조 수준에 따라 surface container color role을 다음과 같이 구분할 수 있다. 

- Surface container lowest 
- Surface container low
- Surface container (default)
- Surface container high
- Surface container highest

<img width="500" alt="스크린샷 2024-09-09 오후 8 19 36" src="https://github.com/user-attachments/assets/b4e7253a-124b-4985-a69c-b4494bcb507f">

기본적으로 네비게이션 바, 메뉴, 다이얼로그 같은 neutral color 컴포넌트에 특정한 surface container color가 매핑되지만, 이러한 역할은 개발자가 사용자 요구사항에 맞게 조정할 수 있다. 

### Inverse colors 

Inverse color는 컴포넌트에 선택적으로 적용되어 주변 UI의 색과 반대되는 색을 구현하여 대비 효과를 만든다.

- **Inverse surface** : 표면과 대비되는 컴포넌트의 배경 색상 
- **Inverse on surface** : inverse surface 위의 텍스트 및 아이콘 
- **Inverse primary** : Inverse surface와 대비되며, 텍스트 버튼처럼 action이 가능한 컴포넌트의 색상 

<img width="500" src="https://github.com/user-attachments/assets/289a4581-b755-4804-be1e-9a911a7da026">

## Outline 

- **Outline** : 텍스트 필드처럼 중요한 경계선
- **Outline variant** : divider 같은 장식용 컴포넌트  

<img width="600" src="https://github.com/user-attachments/assets/dbd35878-f596-4204-b4fc-d20b4051c770">

# 참고 자료 

https://m3.material.io/styles/color/roles