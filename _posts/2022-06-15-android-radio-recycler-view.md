---
title: "[Android] RadioButton 묶음을 Recycler view로 구현하기"
date: 2022-06-15 17:20:00 -0400
categories: android
tags:
  - android
  - recycler_view
  - radio_button
---

Radio 버튼 묶음을 Recycler view로 만들기 위해 찾아보니 `RadioGroup`을 이용할 수는 없었다. 선택된 인덱스를 저장하고 이것을 활용하는 방식으로 구현해야 했다.

`ViewHolder`에서 `position`에 해당하는 인덱스면 체크된 것으로 표시면 된다. 그리고 클릭 리스너를 통해 클릭시 `notifyItemChanged`를 이용하여 바뀐 항목들을 업데이트한다.
이후에 선택된 데이터를 `ViewModel`에 저장하여 활용하면 된다.

# 미리보기

![preview](/assets/images/android_radio_recycler_view.gif)

# MainActivity.kt

특별한 것 없이 `binding`을 이용하는 모습이다.

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.recyclerview.widget.LinearLayoutManager
import com.foundy.uitest2.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    private var _binding: ActivityMainBinding? = null
    private val binding: ActivityMainBinding get() = requireNotNull(_binding)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        _binding = ActivityMainBinding.inflate(layoutInflater)

        setContentView(binding.root)

        binding.recyclerView.apply {
            adapter = RadioAdapter()
            layoutManager = LinearLayoutManager(this@MainActivity)
        }
    }
}
```

<br>

# RadioAdapter.kt

이전에 선택된 항목의 인덱스와 현재 선택된 인덱스의 항목을 저장하고 있다. `radioButton.isChecked = selectedIndex == position`와 `notifyItemChanged`를 이용하는 모습이 중요하다.

```kotlin
import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView
import com.foundy.uitest2.databinding.RadioItemBinding

class RadioAdapter : RecyclerView.Adapter<RadioAdapter.ViewHolder>() {

    private var lastSelectedIndex = 0
    private var selectedIndex = 0

    inner class ViewHolder(private val binding: RadioItemBinding) :
        RecyclerView.ViewHolder(binding.root) {

        private fun onClick(position: Int) {
            lastSelectedIndex = selectedIndex
            selectedIndex = position

            notifyItemChanged(lastSelectedIndex)
            notifyItemChanged(selectedIndex)
        }

        fun setContent(position: Int) {
            binding.apply {
                textView.text = "Item $position"
                radioButton.isChecked = selectedIndex == position

                root.setOnClickListener { onClick(position) }
                radioButton.setOnClickListener { onClick(position) }
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val layoutInflater = LayoutInflater.from(parent.context)
        val binding = RadioItemBinding.inflate(layoutInflater)
        return ViewHolder(binding)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.setContent(position)
    }

    override fun getItemCount(): Int = 100
}
```

<br>

# actyvity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingHorizontal="16dp" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

<br>

# radio_item.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <RadioButton
        android:id="@+id/radioButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="@color/black"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/radioButton"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="item1" />
</androidx.constraintlayout.widget.ConstraintLayout>
```
