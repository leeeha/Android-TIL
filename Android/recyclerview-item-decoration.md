# 리사이클러뷰 아이템 데코레이션 

리사이클러뷰의 아이템 데코레이션을 커스텀 하는 방법에 대해 알아보자!

## LinearLayoutManager

```kotlin
class MarginItemDecoration(private val spaceSize: Int) : RecyclerView.ItemDecoration() {
    // 리사이클러뷰 아이템 주위의 여백을 설정하는 함수 
    override fun getItemOffsets( 
        outRect: Rect, // 아이템의 사각형 영역 
        view: View, // 아이템 뷰 
        parent: RecyclerView, 
        state: RecyclerView.State 
    ) {
        with(outRect) {
            // 첫번째 아이템에 대해서만 위쪽 여백 추가
            if (parent.getChildAdapterPosition(view) == 0) {
                top = spaceSize
            }
						
            // 그외의 아이템들에 대한 여백 추가 
            left = spaceSize
            right = spaceSize
            bottom = spaceSize
        }
    }
}
```

## GridLayoutManager

`GridLayoutManager`는 RecyclerView의 레이아웃 매니저 중 하나로, 아이템을 그리드 형태로 배치합니다. 

방향이 수직(`GridLayoutManager.VERTICAL`)인 경우, `GridLayoutManager`는 아래로 향하는 열을 형성하며, 아이템들은 가로 방향으로 나란히 배치됩니다. 예를 들어, `spanCount`가 3인 경우에는 아이템들이 3개씩 한 행에 나란히 배치되는 것이죠. 그리고 이어서 다음 행이 시작되며, 이런 식으로 아래로 계속 나아갑니다. 

반면에 방향이 수평(`GridLayoutManager.HORIZONTAL`)인 경우, `GridLayoutManager`는 오른쪽으로 향하는 행을 형성하며, 아이템들은 세로 방향으로 나란히 배치됩니다. `spanCount`가 3인 경우에는 아이템들이 3개씩 한 열에 나란히 배치되고, 그 옆으로 다음 열이 시작되는 방식입니다.

<img width="500" src="https://github.com/leeeha/Android-TIL/assets/68090939/8f630d7e-9dad-441e-a3be-42a5cb9a8290"/>

<br>

```kotlin
class MarginItemDecoration(
    private val spaceSize: Int,
    private val spanCount: Int = 1,
    private val orientation: Int = GridLayoutManager.VERTICAL
) : RecyclerView.ItemDecoration() {
    override fun getItemOffsets(
        outRect: Rect, 
        view: View,
        parent: RecyclerView, 
        state: RecyclerView.State
    ) {
        with(outRect) {
            if (orientation == GridLayoutManager.VERTICAL) {
                // 첫번째 행에만 위쪽 여백 추가 
                if (parent.getChildAdapterPosition(view) < spanCount) {
                    top = spaceSize
                }

                // 첫번째 열에만 왼쪽 여백 추가 
                if (parent.getChildAdapterPosition(view) % spanCount == 0) {
                    left = spaceSize
                }
            } else {
                // 첫번째 열에만 왼쪽 여백 추가 
                if (parent.getChildAdapterPosition(view) < spanCount) {
                    left = spaceSize
                }

                // 첫번째 행에만 위쪽 여백 추가 
                if (parent.getChildAdapterPosition(view) % spanCount == 0) {
                    top = spaceSize
                }
            }

            right = spaceSize
            bottom = spaceSize
        }
    }
}
```

## 아이템 간의 마진값 적용

```kotlin
// 리니어 레이아웃 매니저 
recyclerView.addItemDecoration(
    MarginItemDecoration(resources.getDimensionPixelSize(R.dimen.margin))
)

// 그리드 레이아웃 매니저 
recyclerView.addItemDecoration(
    MarginItemDecoration(
        spaceSize = resources.getDimensionPixelSize(R.dimen.margin),
        spanCount = GRID_SPAN_COUNT
    )
)
```

## 참고자료

[A better way to set RecyclerView items margin](https://cesarmorigaki.medium.com/a-better-way-to-set-recyclerview-items-margin-708ea9d3ac25)

[안드로이드 크기 리소스 관리 (dimen)](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=sangrime&logNo=220591538900)
