# Vanilla Vue と TypeScript Vue の比較

## 資料概要
* Vanilla Vue と TypeScript Vue の比較
* Vue の機能一覧

## 用語
* この資料で使用される用語「インスタンス」は「コンポーネント」と同義語です。

## Vanilla Vue
``` html
<script>
import sampleComponent from '~/components/sample-component.vue';

export default {
  // このインスタンスで使用するコンポーネントの定義
  components: {
    sampleComponent,
  },
  // このインスタンスで使用するデータ定義
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
  // 算出プロパティーの定義
  computed: {
    // getter関数 {{ testFn }} / this.testFn　で実行される
    testFn: function() {
      return this.testInfo;
    },
    // getter/setter関数の定義 
    sampleFn: {
      get: function() {
        return this.sampleInfo;
      },
      // this.sampleFn = 'hoge' で実行される
      set: function(val) {
        this.sampleInfo = val;
      }
    }
  },
  // メソッドの定義
  methots: {
    // {{ sampleFn() }} / this.sampleFn() で実行される
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
## TypeScript Vue
``` ts
<script lang="ts">
import sampleComponent from '~/components/sample-component.vue';
// nuxt-property-decorator は watch, props などの機能をデコレータとして使用するために使用
import { Component, Prop, Vue, Watch } from 'nuxt-property-decorator';

// component: {}
@Component({
  components: {
    sampleComponent,
  }
})
export default class extends Vue {

  // data() { return {} }
  private sampleInfo: string = 'string';
  private testInfo: string = 'hello'

  // props: []
  @Props()
  public aaa?: string;
  @Props()
  public bbb?: string;
  @Props()
  public ccc?: string;

  // computed: {}
  get testFn(): string {
    return this.testInfo;
  }
  get sampleFn(): string {
    return this.sampleInfo;
  }
  set sampleFn(val) {
    this.sampleInfo = val;
  }

  // methots: {}
  private sampleFn(): void {
    console.log('test');
  }

  // watch: {}
  @Watch('sampleFn')
  private sampleFnHandler() {
    console.log('sampleFn に変更があったら実行される');
  }

  // ▼ 各ライフサイクルメソッド
  private beforeCreate() {
  }

  private created() {
  }

  private beforeMount() {
  }

  private mounted() {
  }

  private beforeUpdate() {
  }

  private updated() {
  }

  private beforeDestroy() {
  }

  private destroyed() {
  }
}
</script>
```
## 参考資料
* [Vue.jsのライフサイクルメモ_Qiita_171016](https://qiita.com/kurosame/items/6ab7622fe30c299a693e)
* [nuxt-property-decorator_npmDocument](https://www.npmjs.com/package/nuxt-property-decorator)
* [create-nuxt-appで構築したNuxt2のプロジェクトにTypeScriptを導入_Qiita_181004](https://qiita.com/TsukasaGR/items/5c911a59875dce87fe42)