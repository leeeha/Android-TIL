프래그먼트를 생성할 때 자주 하는 실수가 있다. 그건 바로 **기본 생성자를 만들지 않는 것**이다. 

일반적인 경우에는 문제가 안 되겠지만, 화면 회전 등으로 **Configuration Change가 발생**하면 액티비티가 파괴되었다가 재생성 되며 그 안에 있는 **프래그먼트도 재생성** 된다. 이때, 프래그먼트의 기본 생성자를 만들어주지 않으면 다음과 같은 런타임 에러가 발생할 수 있다.

>Caused by: androidx.fragment.app.Fragment$**InstantiationException**: 
Unable to instantiate fragment com.runnect.runnect.util.custom.CommonDialogFragment: 
**could not find Fragment constructor** at androidx.fragment.app.Fragment.instantiate(Fragment.java:678)

>Caused by: java.lang.**NoSuchMethodException**: 
com.runnect.runnect.util.custom.CommonDialogFragment.<init> []
at java.lang.Class.**getConstructor0**(Class.java:2363)
at java.lang.Class.**getConstructor**(Class.java:1759)

위와 같은 에러를 방지하려면, newInstance() 함수에서 프래그먼트의 기본 생성자를 정의해주면 된다. 

```kotlin

class SomeFragment : Fragment(){
	// ... 

  companion object {
	  @JvmStatic
    fun newInstance() = SomeFragment().apply {
		    arguments = Bundle().apply { 
	        putString("key", "value")
	      }
    }
  }
}
```

## 다이얼로그 프래그먼트 예시

### AS-IS

```kotlin
package com.runnect.runnect.util.custom

import android.os.Bundle
import android.view.View
import com.runnect.runnect.R
import com.runnect.runnect.binding.BindingDialogFragment
import com.runnect.runnect.databinding.FragmentCommonDialogBinding

class CommonDialogFragment(
    private val description: String,
    private val negativeButtonText: String,
    private val positiveButtonText: String,
    val onNegativeButtonClicked: () -> Unit,
    val onPositiveButtonClicked: () -> Unit
) : BindingDialogFragment<FragmentCommonDialogBinding>(R.layout.fragment_common_dialog) {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        dialog?.setCanceledOnTouchOutside(false)
        initDialogText()
        initNegativeButtonClickListener()
        initPositiveButtonClickListener()
    }

    private fun initDialogText() {
        with(binding){
            tvCommonDialogDesc.text = description
            btnCommonDialogNegative.text = negativeButtonText
            btnCommonDialogPositive.text = positiveButtonText
        }
    }

    private fun initNegativeButtonClickListener() {
        binding.btnCommonDialogNegative.setOnClickListener {
            onNegativeButtonClicked.invoke()
            dismiss()
        }
    }

    private fun initPositiveButtonClickListener() {
        binding.btnCommonDialogPositive.setOnClickListener {
            onPositiveButtonClicked.invoke()
            dismiss()
        }
    }
}
```

### TO-BE

```kotlin
package com.runnect.runnect.util.custom.dialog

import android.os.Bundle
import android.view.View
import com.runnect.runnect.R
import com.runnect.runnect.binding.BindingDialogFragment
import com.runnect.runnect.databinding.FragmentCommonDialogBinding
import com.runnect.runnect.util.extension.getCompatibleParcelableExtra

class CommonDialogFragment
    : BindingDialogFragment<FragmentCommonDialogBinding>(R.layout.fragment_common_dialog) {
    private var onNegativeButtonClicked: () -> Unit = {}
    private var onPositiveButtonClicked: () -> Unit = {}

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        dialog?.setCanceledOnTouchOutside(false)
        initDialogText()
        initNegativeButtonClickListener()
        initPositiveButtonClickListener()
    }

    private fun initDialogText() {
        binding.dialogText = arguments?.getCompatibleParcelableExtra(ARG_COMMON_DIALOG)
    }

    private fun initNegativeButtonClickListener() {
        binding.btnCommonDialogNegative.setOnClickListener {
            onNegativeButtonClicked.invoke()
            dismiss()
        }
    }

    private fun initPositiveButtonClickListener() {
        binding.btnCommonDialogPositive.setOnClickListener {
            onPositiveButtonClicked.invoke()
            dismiss()
        }
    }

    companion object {
        private const val ARG_COMMON_DIALOG = "common_dialog_params"

        @JvmStatic
        fun newInstance(
            commonDialogText: CommonDialogText,
            onNegativeButtonClicked: () -> Unit,
            onPositiveButtonClicked: () -> Unit
        ) = CommonDialogFragment().apply {
            arguments = Bundle().apply {
                putParcelable(ARG_COMMON_DIALOG, commonDialogText)
            }
            this.onNegativeButtonClicked = onNegativeButtonClicked
            this.onPositiveButtonClicked = onPositiveButtonClicked
        }
    }
}
```

## 참고자료

[[Android] Fragment 생성시 newInstance()를 쓰는 이유](https://medium.com/kenneth-android/fragment-생성시-newinstance-를-쓰는-이유-4571cc0f7760)