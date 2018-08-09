* Android7.0以后进入WebView之后会导致多语言设置被更改。通过以下方法进行适配：
```
    /**
     * <p>解决Android7.0以上进入WebView会更改系统语言的BUG</p>
     * <p>https://stackoverflow.com/questions/40398528/android-webview-language-changes-abruptly-on-android-n</p>
     */
    private void initLocale() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N
            && GlobalHolder.sCurrentLocale != null) {
            // locale为null时setDefault会报NEP
            Locale.setDefault(GlobalHolder.sCurrentLocale);
            final Resources resources = getResources();
            final Configuration config = resources.getConfiguration();
            config.setLocale(GlobalHolder.sCurrentLocale);
            resources.updateConfiguration(config,
                    resources.getDisplayMetrics());
        }
    }
    
    // 然后在使用了WebView的页面的onPause方法中调用该方法。这样从webview退出或从webview进入别的Activity都会重新设置当前的语言

    @Override
    public void onPause() {
        initLocale();
    }

```