# 리사이클러뷰의 등장 배경

RecyclerView 이전에는 스크롤 되는 리스트를 표현하기 위해 ListView를 사용했다. 

ListView는 간단하고 빠르게 만들 수 있다는 게 장점이지만, 다음과 같은 단점들도 존재한다. 

- 기본적으로 애니메이션 기능을 제공하지 않으므로, 개발자가 직접 구현해야 한다.
- 수직 방향의 스크롤만 지원한다.

<details>
<summary>스크롤 시 버벅이는 현상이 발생할 수 있다.</summary>
<div markdown="1">
Layout을 inflate할 때 뷰가 하나밖에 없다면 해당 View를 찾는 과정을 한번만 하면 된다. 하지만 View가 Group을 이루고 있다면 (ViewGroup) <b>그 아래의 모든 childrenView에 대해서 findViewById()를 호출하며, 해당 뷰가 맞는지 확인하는 과정</b>을 거친다.
ViewGroup이 가진 뷰의 개수에 비례하여 findViewById() 연산 작업이 반복되므로, 이 과정에서 성능 저하가 발생할 수 있다. <b>리스트 뷰는 데이터 목록의 전체 크기만큼 아이템 뷰를 생성하기 때문에, 뷰를 찾는 연산 과정이 더 오래 걸린다.</b>
</div>
</details>

<br>

이런 문제점을 해결하기 위해 리사이클러뷰가 등장했다. 

![image](https://github.com/leeeha/Android-TIL/assets/68090939/13c26c63-213e-436a-b301-042b2f2af580)

리사이클러뷰는 데이터 목록을 아이템 뷰 단위로 구성하여 출력하는 ViewGroup이며, 이것을 사용하는 가장 큰 이유는 **재사용성**이 좋기 때문이다. 

ListView는 전체 데이터 크기만큼 아이템 뷰를 생성하며, 이를 재활용하지 않는다. 

반면에, **RecyclerView는 현재 화면에 보이는 뷰 객체만 생성하며, 스크롤로 인해 더 이상 보이지 않게 된 뷰 객체를 재활용** 한다. 따라서 뷰의 생성과 삭제가 매번 반복되는 것보다 성능과 비용 측면에서 훨씬 효율적이다. 

이를 가능하게 만드는 리사이클러뷰의 구조에 대해 알아보자. 

# 리사이클러뷰의 구조

### 뷰홀더 (ViewHolder)

**화면에 표시될 아이템 뷰를 저장하는 객체**

- 리사이클러뷰는 현재 화면에 보이는 뷰 객체만 생성하며, 스크롤로 인해 더 이상 보이지 않게 된 뷰 객체를 재활용한다. 이를 위해서는 **아이템 뷰를 기억하고 있을 객체**가 필요하며, 이것이 바로 ViewHolder이다.
- 뷰홀더가 아이템 뷰를 갖고 있고, 어댑터에 의해 바인딩 된 데이터가 화면에 표시된다. **스크롤 시 이미 만들어진 아이템 뷰를 재활용할 수 있으면, 데이터만 바인딩 시켜 화면에 표시**한다.
- 리스트 뷰에서도 성능 개선을 위해 개발자가 직접 뷰홀더 패턴을 적용할 수도 있다.

### 어댑터 (Adapter)

**데이터 목록을 아이템 뷰에 바인딩 시키는 역할**

- 어댑터의 사전적 정의: 호환되지 않는 장치를 서로 ‘연결’시키기 위한 보조 기구
- 뷰홀더 객체를 생성하고, 해당 뷰홀더가 갖고 있는 **아이템 뷰에 데이터를 바인딩** 시킨다.
- getItemCount(), onCreateViewHolder(), onBindViewHolder()

### 레이아웃 매니저 (LayoutManager)

**아이템 뷰가 화면에 배치되는 형태 관리** (Vertical, Horizontal, Grid, Staggered)

![image](https://github.com/leeeha/Android-TIL/assets/68090939/ae828ed2-ccce-4cdd-a0b0-b5a99ef3fe24)

# 리사이클러뷰 구현 순서

- 액티비티/프래그먼트 xml에 `androidx.recyclerview.widget.RecyclerView` 추가
- 리사이클러뷰의 `아이템 뷰` 레이아웃 구현
- 리사이클러뷰의 `어댑터`, `뷰홀더` 클래스 구현
- 액티비티/프래그먼트에서 `어댑터, 레이아웃 매니저, 아이템 데코레이션` 지정

# 스크롤뷰로 감싸면 안 되는 이유

리사이클러뷰의 작동 원리를 다시 생각해보자. 

1. 아이템 뷰로 채워진 스크롤 가능한 영역이 있다.

![image](https://github.com/leeeha/Android-TIL/assets/68090939/31bdb015-66ae-4492-9280-e81668df3ba4)

2. 스크롤 해서 더 이상 보이지 않는 아이템을 리사이클러뷰는 컴포넌트로부터 꺼낸다. 

![image](https://github.com/leeeha/Android-TIL/assets/68090939/120219fe-a828-429d-8f13-b392d4039772)

3. 꺼내진 아이템 뷰는 스크롤에 의해 앞으로 보여질 또 다른 아이템 뷰에 사용될 수 있다. 

![image](https://github.com/leeeha/Android-TIL/assets/68090939/5b6c732c-8b8b-407b-8e9d-78a3c7a7f62f)

아래 보이는 코드처럼 **리사이클러뷰를 스크롤 뷰로 감싸면, 리사이클러뷰의 크기는 모든 아이템 뷰의 크기가 된다. 결과적으로 리사이클러뷰 자체는 더 이상 스크롤 할 수 없으며, 어떤 아이템도 재활용 되지 않는다.**

```xml
<android.support.v4.widget.NestedScrollView
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <ImageView
           android:layout_width="match_parent"
           android:layout_height="100dp"
           android:scaleType="centerCrop"
           android:src="@drawable/test_image"/>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="#3F51B5"
            android:text="Example"/>

        <android.support.v7.widget.RecyclerView
            android:id="@+id/recycler"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"/>
    </LinearLayout>
</android.support.v4.widget.NestedScrollView>
```

이때 리사이클러뷰는 전체 영역을 화면에 보이는 영역으로 간주하며, 모든 아이템들은 매순간 이용 가능해야 한다. 즉, 리사이클러뷰가 레이아웃 계층 구조에 있는 한, **모든 아이템 뷰들은 메모리에 유지**되는 것이다. 

이는 **스크롤 뷰에 모든 아이템 뷰를 수동으로 추가하는 것과 성능이 동일하며, 리사이클러뷰로 인한 성능 향상은 기대할 수 없다.**

<img width="400" src="https://github.com/leeeha/Android-TIL/assets/68090939/e007d22d-8379-4de5-b4a6-caf1821998d6"/>

👉 리사이클러뷰를 스크롤 뷰로 감싸면, 전체 데이터 크기만큼 아이템 뷰가 생성된다. 

<img width="400" src="https://github.com/leeeha/Android-TIL/assets/68090939/2ed049b0-b12f-4258-959b-84bb78803c36">

👉 리사이클러뷰는 현재 화면에 보이는 아이템 뷰만 생성하고, 필요에 따라 이를 재활용한다. 

# 요약 

> **Using this pattern, the recycler pattern doesn’t work. all the views will be loaded at once** because wrap_content needs the height of complete RecyclerView so it will draw all child Views at once. No view will be recycled and as a result the performance will be so bad. 
Try not to use this pattern unless there are few items in the list, maybe less than eight.

이 패턴을 사용하면, 재활용 패턴이 작동하지 않습니다. **wrap_content는 완전한 리사이클러뷰의 높이가 필요하므로 모든 자식 뷰를 한 번에 그립니다. 어떤 뷰도 재활용되지 않으며 결과적으로 성능이 매우 나빠집니다.** 이 패턴은 목록에 항목이 적을 때 (약 8개 미만일 때만) 사용하려고 노력하세요. 

# 대안은?

- ConcatAdapter
- MultiView Type Adapter 
- 등등 

# 참고자료

[안드로이드 리사이클러뷰 기본 사용법. (Android RecyclerView)](https://recipes4dev.tistory.com/154)

[[Android] RecyclerView(1) - 기본 사용법](https://velog.io/@hyeryeong/RecyclerView-잘-사용하기1)

[[Android] ListView와 RecyclerView Adapter](https://velog.io/@heetaeheo/ListView와-RecyclerView)

[[번역] RecyclerView를 Wrapping했을 때 퍼포먼스 이슈 - feat.NestedScrollView](https://yk-coding-letter.tistory.com/30)

[[Android] NestedScrollView에 대해 알아보자!](https://velog.io/@kimbsu00/Android-7)