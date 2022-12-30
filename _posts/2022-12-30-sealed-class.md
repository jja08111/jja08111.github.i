---
title: "[Kotlin] sealed class"
date: 2022-12-30 01:00:00 +0900
categories: kotlin
tags:
  - kotlin
  - sealed_class
---

Kotlin의 sealed class는 계층 구조를 안전하게 만들어준다.
즉, sealed class에 상속된 sub class들이 무엇이 있는 지 컴파일 타임에 알 수 있다. 이 말이 무슨 뜻일지 파헤쳐보자.

먼저 abstract class와 이를 일반화하는 클래스들을 살펴보자. `when` 구문을 이용하면 else 분기가 꼭 필요하다. 없다면 컴파일러가 에러를 보인다.

```kotlin
abstract class Foo

class Bar : Foo()
class Baz : Foo()

// ...

fun render(foo: Foo) {
  // COMPILE ERROR!
  val some = when (foo) { // -> 'when' expression must be exhaustive, add necessary 'else' branch
    is Bar -> { /* ... */ }
    is Baz -> { /* ... */ }
  }
}
```

이와 다르게 sealed class는 상속된 sub class들이 무엇이 있는 지 컴파일 타임에 알 수 있기 때문에 아래와 같은 구문이 가능하다.

```kotlin
sealed class Foo {
  class Bar: Foo()
  class Baz: Foo()
}

// ...

fun render(foo: Foo) {
  val some = when (foo) {
    is Foo.Bar -> { /* ... */ }
    is Foo.Baz -> { /* ... */ }
  }
}
```

그렇다면 enum class와 비교하였을 때 무엇이 다를까? 먼저 enum은 각 값들이 상수 값들이다. 이는 enum 클래스가 상속할 수 없다는 것을 의미하고 인스턴스를 생성할 수 없다는 것을 의미한다.

```kotlin
enum class Foo {
    BAR,
    BAZ
}

// COMPILE ERROR!
class Child : Foo() // This type is final, so it cannot be inherited from

fun some() {
  // COMPILE ERROR!
  val foo = Foo() // Enum types cannot be instantiated
}
```

이와 다르게 sealed class는 **자식 클래스들의 인스턴스를 만들 수 있으며 동일한 패키지 내에서 상속이 가능**하다.

```kotlin
sealed class Foo {
    open class Bar: Foo()
}

class Baz : Foo.Bar()

fun some() {
  val bar = Foo.Bar()
}
```

# Usage

이 sealed 클래스가 유용하게 사용되는 곳은 아래의 두 경우이다.

- UiState를 sealed class로 나타내는 경우
- RecyclerView의 Adapter에 다양한 타입을 제공하는 경우

## UiState

MVVM 패턴을 이용할 때 UiState를 sealed class를 이용하여 구현할 수 있다.
(아래의 구조는 각 상태가 변경되면 이전 상태를 별도로 보관하지 않는 한 이전 상태에 대한 데이터를 복구할 방법이 없다는 점이 단점이다.
https://medium.com/@laco2951/android-ui-state-modeling-어떤게-좋을까-7b6232543f25)

```kotlin
sealed class MainUiState {
  object Loading : MainUiState()
  data class Success(val name: String) : MainUiState()
  data class Error(val exception: Throwable) : MainUiState()
}
```

위의 클래스를 이용하여 뷰에서는 아래와 같이 처리할 수 있다.

```kotlin
fun updateUi(uiState: MainUiState) {
  when (uiState) {
    is MainUiState.Success -> {
      // ...
    }
    is MainUiState.Error -> {
      // ...
    }
    Loading -> {
      // ...
    }
  }
}
```

## Recycler View

리사이클러 뷰에 하나의 뷰홀더가 아닌 다양한 타입의 뷰홀더가 필요한 경우 sealed class가 제격이다.
아래와 같이 `PostUiModel`, `PostViewHolder`를 sealed class를 이용하면 타입을 안정적으로 구현할 수 있다.

```kotlin
sealed class PostUiModel(val viewType: Int) {

    companion object {
        const val HEADER_VIEW_TYPE = 0
        const val ITEM_VIEW_TYPE = 1
        const val FOOTER_VIEW_TYPE = 2
    }

    data class Header(val title: String) : PostUiModel(HEADER_VIEW_TYPE)

    data class Item(val title: String, val author: String) : PostUiModel(ITEM_VIEW_TYPE)

    data class Footer(val copyright: String) : PostUiModel(FOOTER_VIEW_TYPE)
}

val posts: List<PostUiModel> = listOf(
    PostUiModel.Header(title = "공지사항"),
    PostUiModel.Item(title = "1월 15일 모임", author = "김민성"),
    PostUiModel.Item(title = "1월 8일 모임", author = "김민성"),
    PostUiModel.Item(title = "1월 1일 모임", author = "김민성"),
    PostUiModel.Header(title = "추천목록"),
    PostUiModel.Item(title = "Jetpack Compose 탐험", author = "김민성"),
    PostUiModel.Item(title = "RecyclerView 파헤치기", author = "김민성"),
    PostUiModel.Footer(copyright = "jja08111")
)

sealed class PostViewHolder(
    binding: ViewBinding
) : ViewHolder(binding.root) {

    class Header(
        private val binding: ItemHeaderBinding
    ) : PostViewHolder(binding) {

        fun bind(item: PostUiModel.Header) {
            binding.headerTitle.text = item.title
        }
    }

    class Item(
        private val binding: ItemPostBinding
    ) : PostViewHolder(binding) {

        fun bind(item: PostUiModel.Item) {
            binding.title.text = item.author
            binding.author.text = item.author
        }
    }

    class Footer(
        private val binding: ItemFooterBinding
    ) : PostViewHolder(binding) {

        fun bind(item: PostUiModel.Footer) {
            binding.copyright.text = item.copyright
        }
    }
}

class PostAdapter : ListAdapter<PostUiModel, PostViewHolder>(DIFF) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PostViewHolder {
        return when (viewType) {
            PostUiModel.HEADER_VIEW_TYPE -> PostViewHolder.Header(
                ItemHeaderBinding.inflate(
                    LayoutInflater.from(parent.context),
                    parent,
                    false
                )
            )
            PostUiModel.ITEM_VIEW_TYPE -> PostViewHolder.Item(
                ItemPostBinding.inflate(
                    LayoutInflater.from(parent.context),
                    parent,
                    false
                )
            )
            PostUiModel.FOOTER_VIEW_TYPE -> PostViewHolder.Footer(
                ItemFooterBinding.inflate(
                    LayoutInflater.from(parent.context),
                    parent,
                    false
                )
            )
            else -> throw  IllegalArgumentException()
        }
    }

    override fun onBindViewHolder(holder: PostViewHolder, position: Int) {
        val item = getItem(position)
        when (holder) {
            is PostViewHolder.Header -> holder.bind(item as PostUiModel.Header)
            is PostViewHolder.Item -> holder.bind(item as PostUiModel.Item)
            is PostViewHolder.Footer -> holder.bind(item as PostUiModel.Footer)
        }
    }

    override fun getItemViewType(position: Int): Int {
        return getItem(position).viewType
    }

    companion object {
        private val DIFF = object : DiffUtil.ItemCallback<PostUiModel>() {

            override fun areItemsTheSame(oldItem: PostUiModel, newItem: PostUiModel): Boolean {
                return oldItem == newItem
            }

            override fun areContentsTheSame(oldItem: PostUiModel, newItem: PostUiModel): Boolean {
                return oldItem == newItem
            }
        }
    }
}
```
