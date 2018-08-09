* imeOptions属性可以改变回车键
* actionGo、 actionSend 、actionSearch、actionDone四种可选属性
* 有时候该属性可能无效，设置下面两个属性中的一个即可使这个属性生效：
    * 1 将singleLine设置为true
    * 2 将inputType设置为text 
* 监听回车按钮事件

```
EditText editText = (EditText) contentView.findViewById(R.id.editText);  
editText.setOnEditorActionListener(new OnEditorActionListener() {  
    @Override  
    public boolean onEditorAction(TextView v, int actionId,  
            KeyEvent event) {  
        if (actionId == EditorInfo.IME_ACTION_SEARCH) {  
            Toast.makeText(getActivity(), "1111111",Toast.LENGTH_SHORT).show();  
        }  

        return false;  
    }  
});  
```