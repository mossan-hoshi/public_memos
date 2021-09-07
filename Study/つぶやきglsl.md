# つぶやきGLSL概要
- [eWbGL総本山](https://webgl.souhonzan.org/)などの作者[doxasさん](https://twitter.com/h_doxas?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor)によるTwitterに投稿可能なGLSLコード作成サイト
- [ホームページ](https://twigl.app/) / [GitHub](https://github.com/doxas/twigl)
- 解説
  - 公式[1](https://webgl.souhonzan.org/entry/?v=1708),[2](https://webgl.souhonzan.org/entry/?v=1709),[3](https://webgl.souhonzan.org/entry/?v=1710),[4](https://webgl.souhonzan.org/entry/?v=1711),[5](https://webgl.souhonzan.org/entry/?v=1712)
  - [つぶやきGLSLのススメ](https://www.slideshare.net/yutakasato391/glsl-249579645)
  - [つぶやきGLSLで今すぐ使えるシェーダーminifyテクニック11選！](https://notargs.hateblo.jp/entry/twigl_minify)
  - [今日から使えるつぶやきGLSLのハック集](https://scrapbox.io/sayachang/%E4%BB%8A%E6%97%A5%E3%81%8B%E3%82%89%E4%BD%BF%E3%81%88%E3%82%8B%E3%81%A4%E3%81%B6%E3%82%84%E3%81%8DGLSL%E3%81%AE%E3%83%8F%E3%83%83%E3%82%AF%E9%9B%86)

# モードによる違い

## 主な違い
| 意味                   | classic | geek  | geeker  | geekest |
| :--------------------- | :-----: | :---: | :-----: | :-----: |
| メイン関数             |   〇    |  〇   |   〇    | ✖(なし) |
| 外部変数/precision定義 |   〇    |  〇   | ✖(なし) | ✖(なし) |
## 変数・関数名の違い
### standard
| 意味               |   classic    |     geek     |    geeker    |   geekest    |
| :----------------- | :----------: | :----------: | :----------: | :----------: |
| ディスプレイ解像度 |  resolution  |      r       |      r       |      r       |
| マウス座標         |    mouse     |      m       |      m       |      m       |
| 経過時間           |     time     |      t       |      t       |      t       |
| 前フレーム                  |  backbuffer  |      b       |      b       |      b       |
| 画素座標           | gl_FragCoord | gl_FragCoord | gl_FragCoord |      FC      |
| 出力画素値         | gl_FragColor | gl_FragColor | gl_FragColor | gl_FragColor |

### geek
| 意味               |   stadard    |    300 es    |  MRT  |
| :----------------- | :----------: | :----------: | :---: |
| ディスプレイ解像度 |  resolution  |      ←       |       |
| マウス座標         |    mouse     |      ←       |       |
| 経過時間           |     time     |      ←       |       |
| ?                  |  backbuffer  |      ←       |       |
| 出力画素値         | gl_FragCoord | gl_FragCoord |       |
| 出力画素値         | gl_FragColor |   outColor   |       |

### geeker
| 意味               | stadard      | 300 es       | MRT  |
| :----------------- | :----------- | :----------- | :--- |
| ディスプレイ解像度 | r            |              |      |
| マウス座標         | m            |              |      |
| 経過時間           | t            |              |      |
| ?                  | b            |              |      |
| 出力画素値         | gl_FragCoord | gl_FragCoord |      |
| 出力画素値         | gl_FragColor | outColor     |      |

### geekest
| 意味               | stadard | 300 es | MRT  |
| :----------------- | :------ | :----- | :--- |
| ディスプレイ解像度 | r       |        |      |
| マウス座標         | m       |        |      |
| 経過時間           | t       |        |      |
| ?                  | b       |        |      |
| 出力画素値         | FC      | FC     |      |

# デフォルトシェーダーの挙動(mode=classic)
```glsl
// つぶやきglslは「精度定義／外部入力定義／main関数」の3要素から成る
/////////////////
// 1.「精度定義」
precision highp float; // ビット制度の定義(この場合は64bit float?)

/////////////////
// 2.「外部入力定義」
uniform vec2 resolution;  // 解像度(例えばフルHD(横幅1920,縦幅1080)の場合はresolution=(1920,1080)
uniform vec2 mouse; // マウス座標(例えばマウスがx=100,y=200の座標にいる場合、mouse=(100,200))
uniform float time; // 経過時間(1秒間=1.0？)

/////////////////
// 3. 「main関数」
void main(){ // 
  vec2 r=resolution; // 短い名前の変数に置き換え
  vec2 p=(gl_FragCoord.xy*2.-r)/min(r.y,r.x)-mouse; // 座標の正規化＆マウス対応
  for(int i=0;i<8;++i){ // フラクタル処理
    vec2 abs_p = abs(p); // 点対象化
    float dot_p = dot(p,p); // 内積(x^2+y^2の放物線)

    float offset = 0.9; // 振幅オフセット
    float gain = cos(time*.2)*.4; // -0.4～0.4のcos(=sin)波(周期は31.415秒)
  
    p.xy=abs_p/dot_p-vec2(offset+gain); // 
  }

  // 出力画素値設定(R=x,G=x,B=y, αは何を入れても違いなし)
  gl_FragColor=vec4(p.xxy,1);
}
```

# 関数
- dot: 内積
  - dot(p)
  - p.x*p.x + p.y*p.y + p.z*p.z + p.a*p.a [pがVec4の場合]
- abs: 絶対値
- min/max
- clamp
- fract=clamp(val,0.0,1.0)
- floor
# Tips・ルール
- 値は0.0~1.0でクリップされる
- 座標の正規化は主に3通り
  1. FC.xy / r;               //  0.0~1.0
  2. FC.xy / r * 2.0 - 1.0;   // -1.0~1.0
  3. (FC.xy * 2.0 - r)/ min(r.x,r.y) // 短辺において -1.0~1.0
    - 画面によってアスペクト比が変わるのを防ぐ
