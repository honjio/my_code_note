# Vue と TypeScript の比較 
``` html
<script>
import sampleComponent from '~/components/sample-component.vue';

export default {
  // このコンポーネント内で使用するコンポーネントの定義
  components: {
    sampleComponent,
  },
  // 状態を管理する変数
  data() {
    return {
      sampleInfo: 'string',
      testInfo: 'hello',
    }
  },
  // 親コンポーネントから受け取るデータ定義
  props: [
    'aaa',
    'bbb',
    'ccc',
  ],
  computed: {
    // getter関数
    testFn: function() {
      return this.testInfo;
    },
    // getter/setter関数の定義
    sampleFn: {
      get: function() {
        return this.sampleInfo;
      },
      set: function(val) {
        this.sampleInfo = val;
      }
    }
  },
  methots: {
    sampleFn: function() {
      console.log('test');
    }
  },
  watch: {
    sampleFn: function(val) {
      console.log('sampleFn に変更があったら実行される');
    }
  },
  beforeCreate() {
    /*
    インスタンスが初期化されるときに同期的に呼ばれる
    */
  },
  created() {
    /*
    インスタンスが作成された後に同期的に呼ばれる
    データの監視とイベントの初期セットアップが完了した状態
    サーバーサイドからのデータ取得処理はここで行うのが良さそう
    */
  },
  beforeMount() {
    /*
    インスタンスがマウントされる前に呼ばれる
    templateオプションがrender関数にコンパイルされた後に実行される
    （templateオプションが無い場合は、elで指定したouterHTMLをコンパイルする）
    */
  },
  mounted() {
    /*
    インスタンスがマウントされた後に呼ばれる
    DOM要素にアクセスできるようになる（this.$el
    */
  },
  beforeUpdate() {
    /*
    状態を更新し、Virtual DOMが再描画される前に呼ばれる
    つまり、beforeUpdate内で状態を取得すると更新後の値になっている
    */
  },
  updated() {
    /*
    状態を更新し、Virtual DOMが再描画される後に呼ばれる
    状態変更後のDOM要素にアクセスする場合は、updated内で取得すると良い
    ただし、updated内で状態を更新すると無限ループに陥る可能性がある
    */
  },
  beforeDestroy() {
    /*
    インスタンスが破棄される前に呼ばれる
    この段階ではインスタンスはまだ完全に機能している
    */
  },
  destroyed() {
    /*
    インスタンスが破棄される後に呼ばれる
     */
  }
}
</script>
```