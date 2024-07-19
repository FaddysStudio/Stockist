#!/usr/bin/env roll

# Faddy's Stock Music Production

## Installation

Since this file is a roll, it requires Faddy's Roll to be installed.
In addition, Csound must be installed for this roll to work.

```sh
sudo npm i -g @faddys/roll
```

## Usage

## Implementation

### `.FaddysStockist/orc/`

```roll
?# rm -fr orc ; mkdir orc
```

### `.FaddysStockist/orc/instruments.def`

```roll
?# cat - > .FaddysStockist/orc/instruments.mjs
```

```js
//+==

[ 'kit', 'beat', 'out', 'loop' ]
.forEach ( ( instrument, index ) => console .log ( `#define ${ instrument } #${ index + 1 }#` ) );

//-==
```

```roll
?# node .FaddysStockist/orc/instruments.mjs > .FaddysStockist/orc/instruments.def ; rm .FaddysStockist/orc/instruments.mjs
```

#### `.FaddysStockist/orc/index.part`

```roll
?# cat - > .FaddysStockist/orc/index.part
```

```csound
//+==

sr = 48000
ksmps = 32
nchnls = 2
0dbfs = 1

#include "instruments.def"

#include "kit.part"
#include "beat.part"
#include "out.part"
#include "loop.part"

//-==
```

#### `kit.part`

```roll
?# cat - > .FaddysStockist/orc/kit.part
```

```csound
//+==

instr $kit

SKit strget p4
SPath strget p5

iReady chnget SPath

if iReady == 0 then

SSize sprintf "%s/size", SKit
iSize chnget SSize

SIndex sprintf "%s/%d", SKit, iSize

chnset SPath, SIndex
chnset iSize + 1, SSize

chnset 1, SPath

endif

endin

//-==
```

#### `beat.part`

```roll
?# cat - > .FaddysStockist/orc/beat.part
```

```csound
//+==

instr $beat

iSwing random 0, 1

if iSwing > p5 then

SKit strget p4
SSize sprintf "%s/size", SKit
iKit chnget SSize

iIndex random 0, iKit - 1
SIndex sprintf "%s/%d", SKit, iIndex

SNote chnget SIndex

p3 filelen SNote

kPitch jspline 1, 0, 4

aRhythm [] diskin2 SNote, cent ( kPitch * 10 )

iChannels lenarray aRhythm

if iChannels == 1 then

aLeft = aRhythm [ 0 ]
aRight = aRhythm [ 0 ]

else

aLeft = aRhythm [ 0 ]
aRight = aRhythm [ 1 ]

endif	

kAmplitude jspline .1, 0, 4
kAmplitude += .1
kAmplitude = 1 - kAmplitude

aLeft clip aLeft * kAmplitude, 1, 1
aRight clip aRight * kAmplitude, 1, 1

chnmix aLeft, "left"
chnmix aRight, "right"

endif

endin

//-==

#### `out.part`

```roll
?# cat - > .FaddysStockist/orc/out.part
```

```csound
//+==

instr $out

aLeft chnget "left"
aRight chnget "right"

chnclear "left"
chnclear "right"

aLeft clip aLeft, 1, 1
aRight clip aRight, 1, 1

outs aLeft, aRight

endin

//-==
```

#### `loop.part`

```roll
?# cat - > .FaddysStockist/orc/loop.part
```

```csound
//+==

instr $loop

rewindscore

endin

//-==
```

### `.FaddysStockist/sco/`

?# cat - > .FaddysStockist/sco/

#### `.FaddysStockist/sco/instruments.def`

```roll
?# cp .FaddysStockist/orc/instruments.def .FaddysStockist/sco/
```

#### `.FaddysStockist/sco/index.mjs`

```roll
?# cat - > .FaddysStockist/sco/index.mjs

```js
//+==

import Scenarist from '@faddys/scenarist';
import $0 from '@faddys/command';

try {

await Scenarist ( class Score extends Array {

static index = 0
static tempo = 0
static length = 0

constructor () { super (

'#include "instruments.def"',
'i $out 0 -1',

) }


console .log ( score .join ( '\n\n' ) );

}

score = []
tempo = 105
length = 4
measure = 8

constructor ( file, loop ) {

this .file = file;
this .loop = loop;

}

async $_producer ( $ ) {

const score = this;
const notation = await $0 ( 'cat', score .file )
.then ( $ => $ ( Symbol .for ( 'output' ) ) );

for ( let line of notation )
if ( ( line = line .trim () ) .length )
await $ ( ... line .trim () .split ( /\s+/ ) );

}

$_score () {

const score = this;

Score .length += ( score .length *= 60 / score .tempo );
Score .tempo = score .tempo;

if ( score .loop )
score .score .push ( `i $loop ${ Score .length } 1` );

return score .join ( '\n' );

}

static kit = new Map

async $kit ( $, kit ) {

const score = this;

if ( Score .kit .has ( kit ) ) )
return score .kit = kit;

score .concat ( await $0 ( 'file', '--mime-type', kit + '/*' )
.then ( $ => $ ( Symbol .for ( 'output' ) ) )
.then ( directory => directory .map ( entry => entry .split ( /\s+/ ) )
.filter ( ( [ file, type ] ) => type .startsWith ( 'audio' ) )
.map ( ( [ file ] ) => `i $kit 0 0 "${ kit }" "../${ file .slice ( 0, -1 ) }"` ) ) );

score .kit 

score .kit = kit;

}

$_director ( $, ... argv ) {

const score = this;
const [ step, swing ] = argv .shift () .split ( '/' );

if ( typeof score .start !== 'number' )
score .start = Score .length * Score .tempo / score .tempo;

score .score .push ( [

`i [ $score + .${ ++Score .score % 10 === 0 ? ++ Score .score : Score .score }`,
Score .length + parseFloat ( step ) / score .measure * score .length * 60 / score .tempo,
0,
`"${ score .kit }"`,
parseFloat ( swing ) || 0

] .join ( ' ' ) );

return $ ( Symbol .for ( 'director' ), ... argv );

}

} );

} catch ( issue ) {

console .error ( '@faddys/stockist', issue );

}

//-==
```
?# $ node rhythm.mjs > index.sco

/?# cd .FaddysKit ; csound -odac index.part index.sco
