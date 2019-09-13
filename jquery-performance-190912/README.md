# $('xxx').height( )処理が遅い

## 概要

jQuery の `height()` と vanillaJs での height 取得で速度を比較する

## 調査のきっかけ

下記コードがとあるページで使用されてた。元を辿れば他のページで紹介されていた処理だったので、恐らくそのまま使用されていた。  
要素の高さを超えた分の文字を削る処理だが、問題は処理が遅くページの表示速度に悪影響を及ぼしていた。調査の結果 while の判定式 `$clone.height()` に時間がかかっていた

```js
$('.js-xxxx').each(function () {
    const $target = $(this);
    let html = $target.html();
    const $clone = $target.clone();
    $clone
        .css({
            display: 'none',
            position: 'absolute',
            overflow: 'visible',
        })
        .width($target.width())
        .height('auto');
    $target.after($clone);
    while ((html.length > 0) && ($clone.height() > $target.height())) {
        html = html.substr(0, html.length - 1);
        $clone.html(`${html}...`);
    }
    $target.html($clone.html());
    $clone.remove();
});
```

## 比較方法

下記 script を用いて比較を行う

```js
/**
 * 要素の height の計算を行う。
 * 引数の真偽値で vanillaJs / jQuery での計算に切り替える
 * @param {Boolean} bool
 */
const calculateHeight = (bool) => {
    let totalTime = 0;
    let height = 0;
    for (let i=0; i < 200; i++) {
        const startTime = Date.now();
        const $sample = $('.sample');
        for (let i=0; i < 15; i++) {
            if (bool) {
                const base = $sample[0].clientHeight;
                const top = $sample[0].style.getPropertyValue('padding-top');
                const bottom = $sample[0].style.getPropertyValue('padding-bottom');
                height = base - (parseInt(top) + parseInt(bottom));
            } else {
                height = $sample.height();
            }
        }
        const endTime = Date.now();
        totalTime += endTime - startTime;
    }
    return {
        getHeight() {
            return height;
        },
        getTime() {
            return totalTime;
        }
    }
};
```

## 比較結果

比較 script の結果 jQuery での height 取得が平均 `40~50ms` なのに対し、  
vanillaJs では 平均 `4~5ms`とかなりの差が生じた。
