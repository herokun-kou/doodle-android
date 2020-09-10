# android-doodle
안드로이드 낙서 라이브러리입니다. 그냥 심심풀이 낙서에요.   
지금은 클래스 하나 뿐입니다. 그에 대한 내용은 아래에 있어요.   
물론 스샷같은건 아직 없어서~~올리기 귀찮아서~~ 궁금하시면 직접 import하고 사용해보세요! (약팔이)
   
## AnimatedMenu
동적 애니메이션이 있는 메뉴를 만듭니다.
FrameLayout을 확장하여 화면 전체를 차지했을 때 최적의 퍼포먼스를 발휘하도록 설계되었습니다.   
ConstraintLayout 등을 사용하여 메인 뷰 위에 겹쳐서 전체 화면을 차지하도록 추가하면 완벽합니다.   
show()함수를 호출하면 레이아웃이 차지하는 영역이 어두워(혹은 밝아)지며 화면 중앙에 메뉴 아이템들이 순차적으로 동적 애니메이션과 함께 표시됩니다.   
decoration, header, footer 기능이 있습니다. decoration은 화면이 어두워질 때 같이 페이드되어 나타나는 뷰로, AnimatedMenu 안에 하나의 뷰만 있어야 하지만
ViewGroup도 뷰이기 때문에 여러 뷰를 넣고 싶다면 ViewGroup을 확장한 레이아웃을 추가하고 그 안에 여러 뷰를 넣으면 됩니다.   
header와 footer는 메뉴 아이템들의 가장 위와 가장 아래에 표시되며, 메뉴 아이템들이 animate될 때 같이 animate됩니다.
세 기능 모두 독립적으로 사용할 수 있으며, 사용하지 않을 경우 별다른 설정 없이 그냥 비워두면 됩니다.
사용할 경우 xml attribute를 통해 설정해주세요. 자세한 내용은 아래에 있습니다.
메뉴 아이템들은 그룹을 통해 표시되며, 최소 한 개의 그룹을 가져야합니다. 그룹 안에 화면에 들어가는 한 얼마든지 메뉴 아이템을 추가할 수 있습니다.   
그룹 배열과 그룹 아이템 배열을 지정해주면 되며, 그룹 아이템은 2차원 배열로 length(size)가 그룹 배열과 그룹 아이템 배열이 같아야합니다.   
나머지 내용은 아래의 코드를 살펴보면서 이해해주세요. 
   
### Usage
#### XML
아래의 AnimatedMenu를 하나의 별도 layout파일로 정의하고 <include/> 태그를 통해 다른 레이아웃에서 불러오면 편리합니다.
```xml
<com.herok.doodle.AnimatedMenu
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:useDecoration="true"
    app:useHeader="true"
    app:useFooter="true">

    <!-- First child : Decoration view -->
    <!-- This view is decoration of Menu, it will just fade in/out when menu is show/hides. -->
    <!-- Remove this if you don't want to use decoration and set useDecoration value to false. -->
    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <!-- You can create any view here and layout does not have to be ConstraintLayout. -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintRight_toRightOf="parent"/>

    </androidx.constraintlayout.widget.ConstraintLayout>

    <!-- Second child : MenuItem view -->
    <!-- This view is menu item view, if you don't use header and footer, remove this layout. -->
    <!-- If exists, second child must be LinearLayout. So do not change layout type. -->
    <!-- Remove this layout if you don't use any header and footer. -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <!-- Second child's xml-defined children : Header/Footer view -->
        <!-- This view is header/footer view. Animates with Code-defined menu items. -->
        <!-- If MenuItem view has two children, first child is header view and second child is footer view. -->
        <!-- If MenuItem view has only child and useHeader is true, this view is header. -->
        <!-- If MenuItem view has only child and useFooter is true, this view is footer. -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!"
            android:textColorHint="#FFFFFF"
            android:textSize="30sp"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!"
            android:textColorHint="#FFFFFF"
            android:textSize="30sp"/>

    </LinearLayout>

</com.herok.doodle.AnimatedMenu>
```
주석을 읽어보시면 대강 사용법이 감이 오실거라 생각합니다.
#### Kotlin
뷰의 초기화는 아래와 같이 합니다. 먼저 AnimatedMenu를 가져온 후(ViewBinding으로도 가능합니다) 그룹, 아이템 배열을 만들어서 AnimatedMenu에 setGroupsAndItems를 호출해 설정합니다.   
각 아이템의 클릭 액션은 아래의 setItemClickListener(groupPosition, itemPosition, listener)을 호출해 설정합니다. 이 때 주의하실 점은, 아무 조건 없이 onClickListener 람다식을 채울 경우
여러번 마구 누르면 리스너 액션이 두 번 이상 동작할 수 있다는 점입니다. 한 번만 동작하게 하려면 animating property와 hide() 함수를 사용해주시면 됩니다.   
```kotlin
val animatedMenu: AnimatedMenu = findViewById(R.id.animated_menu)
val animatedMenuGroups = arrayOf("그룹 1", "그룹 2")
val animatedMenuItems = arrayOf(
    arrayOf("아이템 1 - 1", "아이템 1 - 2"),
    arrayOf("아이템 2 - 1", "아이템 2 - 2", "아이템 2 - 3")
)
animatedMenu.setGroupsAndItems(animatedMenuGroups, animatedMenuItems)
animatedMenu.setItemClickListener(1, 2){
    if(!animatedMenu.animating){
        hide()
        //OnClickAction
    }
}
```
   
메뉴를 보이거나 숨기려면 다음 function을 호출합니다.   
```kotlin
animatedMenu.show()
animatedMenu.hide()
```
   
그룹이나 아이템을 아래와 같이 동적으로 추가하거나 제거할 수도 있습니다. 추가/삭제 애니메이션이 필요하다면 animate인자에 true를 전달하시면 됩니다.
그룹을 삭제할 경우 포함된 모든 아이템이 제거되니 주의해야 합니다.
단, 추가/삭제 애니메이션이 진행될 동안은 다른 작업(show, hide, add, remove 등)을 수행할 수 없으니 여러 메뉴를 추가하려 한다면 animate인자를 생략하거나 false로 지정해주세요.
```kotlin
animatedMenu.addGroup(0, "그룹 0", false)    //애니메이션 없이 가장 위에 '그룹 0'을 추가
animatedMenu.addItem(0, 0, "아이템", false){ //애니메이션 없이 첫 번째 그룹에 아이템을 추가
    // OnClickAction
}
animatedMenu.removeItem(0, 0, false)        //애니메이션 없이 첫 그룹의 첫 아이템을 삭제
animatedMenu.removeGroup(0, true)           //애니메이션과 함께 첫 번째 그룹을 삭제
```
   
header와 footer도 동적으로 추가하거나 제거할 수 있습니다.   
단, 이미 header 혹은 footer가 추가된 상태에서 add를 호출하거나 header와 footer가 없는 상태에서 remove를 호출하면 무시되니 주의해야 합니다.
```kotlin
animatedMenu.addHeader(headerView)
animatedMenu.addFooter(footerView)
animatedMenu.removeHeader()
animatedMenu.removeFooter()
```
   
메뉴 아이템의 스타일링은 그룹 텍스트, 아이템 텍스트 전체 단위로 가능합니다. 아이템 하나하나의 스타일을 다르게 지정하는 기능은 없습니다.   
텍스트 색상과 크기가 커스텀 가능하며, animatedMenu의 groupTextColor, itemTextColor, groupTextSize, itemTextSize를 직접 수정하여 반영합니다. 
```kotlin
animatedMenu.itemTextColor = Color.parseColor("#FFFFFF")
animatedMenu.groupTextColor = Color.parseColor("#A0FFFFFF")
animatedMenu.itemTextSize = context.resources.getDimensionPixelSize(R.dimen.item_text_size)
animatedMenu.groupTextSize = context.resources.getDimensionPixelSize(R.dimen.group_text_size)
```

#### Exceptions
AnimatedMenu는 다음 상황에서 Exception을 발생시킵니다:
- Xml 속성 useDecoration 속성이 true이지만 AnimatedMenu의 자식 뷰의 수가 0개일 때
- Xml 속성 useDecoration 속성이 false인데 AnimatedMenu의 자식 뷰의 수가 2개 이상일 때
- Xml 속성 useHeader가 true인데 header view를 찾을 수 없을 때
- Xml 속성 useFooter가 true인데 footer view를 찾을 수 없을 때
물론 이외에도 당연히 Exception은 발생할 수 있으나 대표적인 몇 가지를 나열했습니다.
   
   
## ShootingStar
별똥별 클래스입니다. 화면에 배치하면 해당 뷰가 차지하는 크기 안에서 별똥별이 마구 떨어집니다.   
퀄리티가 좋은 편은 아니고, 그냥 직선이 대각선으로 움직이는게 다입니다. 이것도 궁금하시다면 가져가서 써보세요.   
예전에 웹용 자바스크립트로 짠 코드 재활용해서 안드로이드 버전으로 바꿔보았는데 글쎄요...잘 되었는지는 모르겠네요.
무엇보다 안드로이드는 뷰의 배치나, 뷰 하나가 차지하는 메모리가 무겁다보니 웹처럼 마구 별들을 추가하거나 하지는 못할 것 같습니다.
Xml에서 속성으로 정의된 별의 수 만큼 Star View가 추가되며, 각 Star는 떨어진 후에 위치 딜레이를 거쳐 다시 떨어지게 됩니다.
별이 떨어질 때마다 새로 객체를 만들기엔 낭비일 것 같아서, 정해진 수의 별을 추가하고 그 별들을 재활용하도록 짰습니다.
   
### Usage
#### Xml
아래처럼 뷰를 레이아웃에 추가합니다. 다른 뷰와 겹쳐야 하므로 ConstraintLayout나 FrameLayout 같은걸 쓰는 것을 추천합니다.
```xml
<com.herok.doodle.ShootingStar 
    android:id="@+id/shooting_star"
    android:layout_width="match_parent"
    android:layout_height="200dp"
    app:starCount="5"
    app:minStarDelay="2500"
    app:maxStarDelay="10000"
    app:starRotation="-30"
    app:starColor="#FFFFFF"
    app:shootingSpeed="5"
    app:shootingType="DYNAMIC" />
```
각 옵션들을 설명하면 다음과 같습니다:
- starCount: 한 시점에 최대로 보일 수 있는 별의 수입니다.
- minStarDelay: 한 개의 별이 떨어진 후 재조정을 통해 다시 떨어지기까지의 최소 지연시간입니다. 이 값이 있다고 해서 한 시점에 한 개의 별만 떨어지는 것이 아님에 유의합니다.
- maxStarDelay: 한 개의 별이 떨어진 후 재조정을 통해 다시 떨어지기까지의 최대 지연시간입니다.
- starRotation: 별이 떨어지는 각도입니다. 모든 별이 이 각도를 통해 떨어지며 별 하나하나의 각도를 랜덤하게 부여하는 기능은 아직 없습니다. -90 < 각도 < 90도 이여야 합니다.
- starColor: 별의 색입니다. 반투명하게 하는 것이 조금 더 보기 좋습니다.
- shootingSpeed: 별이 떨어지는 속도입니다. 빠르게 떨어지게 하고 싶다면 값을 크게 주시면 됩니다. 최소 1에서 최대 10의 값을 부여할 수 있습니다.
- shootingType: 떨어지는 형태를 정합니다. STATIC과 DYNAMIC이 있으며, 큰 차이는 없으나 잘 보면 조금 다릅니다. 이건 말로 설명하기가 애매하니 궁금하시면 해보세요!
   
#### Kotlin
Xml에 추가하는 것 만으로 별이 떨어지지는 않습니다. 다음은 코드에서 호출할 수 있는 함수들입니다.
- start() - 별들이 떨어지도록 합니다. 이미 별들이 떨어지고있는 중에 이 함수를 호출하면 무시됩니다.
- stop() - 강제로 모든 별들이 떨어지는 것을 중단합니다. 떨어지고 있는 별이 있을경우 중단되기 때문에 속도가 느릴경우 별이 사라지는게 보일 수 있습니다.
- requestStop() - 별이 떨어지는 것을 멈추도록 뷰에 요청합니다. 딜레이 중인 별들은 딜레이가 취소되며, 떨어지고 있는 별이 있을 경우 떨어지는 것이 끝날때 까지 기다립니다.
- addStars(count: Int) - 별을 추가합니다. 제한을 둔 사항은 없으나 많이 추가하면 퍼포먼스가 떨어지거나 기기에 악영향을 줄 수 있으니, 
minStarDelay나 maxStarDelay를 조정하는 것을 더 추천합니다.
- removeStars(count: Int) - 별을 강제로 제거합니다. 먼저 추가된 별들 부터 순서대로 제거되며, 떨어지고있는 별이 있더라도 상관없이 호출 시점에 모두 제거됩니다.
- requestRemoveStars(count: Int) - 별을 제거하도록 뷰에 요청합니다. 이 경우 별 오브젝트에 특수한 표시를 추가하며, __떨어지는 것이 끝날 때__ 이 표시가 있는 별이 제거됩니다.
딜레이 중인 별들도 제거되지 않으며 무조건 별이 떨어지고 난 직후에 이 표시가 있는지 확인하고 제거되므로 주의해주세요.
- addSize(size: Int) - 별의 크기 값을 리스트에 추가합니다. 별의 크기는 별이 떨어지기 직전에 리스트에 있는 값들 중 랜덤하게 하나를 골라 설정되며, 
기본적으로 50, 150, 250, 350, 450, 550의 값들이 추가되어있습니다.
- clearSize() - 별의 크기 값이 담긴 리스트를 비웁니다. 기본적으로 추가된 값들을 지우고 다른 값들을 넣을 때 호출하시면 됩니다.
   
코드에서는 starCount를 제외한 모든 xml 속성을 같은 이름으로 설정해줄 수 있으며, 설정하면 즉시 반영되므로 별다른 함수를 호출할 필요는 없습니다.
   
### Plus
별 너무 많이 추가하면 처음에는 멋진데 보면볼수록 이상해보이므로 그냥 진짜 별똥별처럼 한두개만 추가하는 것을 권장합니다.   
나름 자원낭비 없이 만드려고 노력하긴 했지만 실패한 부분도 좀 있어서 실제로 쓰기엔 좀 부족할 수 있습니다...


## Conclusion
패키지에 라이브러리 aar파일이 있을겁니다. 흥미가 가신다면 가져가셔서 써보세요.   
코드도 저장소에 있으니 관심있으시면 보시고 마구 까주시면 좋겠습니다. 스파게티거든요.   
어쨌든 그렇습니다! 어디까지나 낙서 프로젝트라 당연히 버그가 많을거에요.
여기까지 보신 분은 없겠지만 둘러봐주셔서 감사합니다.