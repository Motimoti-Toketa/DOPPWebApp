<script setup>
import { ref } from 'vue';
import draggable from 'vuedraggable';
import * as Tone from 'tone';

// ■ 1. データ部分（ドラムパターン）
// 音程(note)の代わりに、sound(音の種類) を保存します
const noteBlocks = ref([
  { id: 1, sound: 'A', color: '#555555' },
  { id: 2, sound: 'B', color: '#555555' },
  { id: 3, sound: 'C', color: '#555555' },
  { id: 4, sound: 'D', color: '#555555' },
  { id: 5, sound: 'E', color: '#555555' },
  { id: 6, sound: 'F', color: '#555555' },
  { id: 7, sound: 'G', color: '#555555' },
  { id: 8, sound: 'H', color: '#555555' },
]);

let nextId = 9;
const isLoaded = ref(false);

// ■ 2. ドラムプレイヤーの準備 (Tone.Players)
// 複数の音源ファイルを「名前」で管理します
const players = new Tone.Players({
  // --- ここに自分のファイルを指定します ---
  "A": "sounds/frag_A2.wav",
  "B": "sounds/frag_B2.wav",
  "C": "sounds/frag_C2.wav",
  "D": "sounds/frag_D2.wav",
  "E": "sounds/frag_E2.wav",
  "F": "sounds/frag_F2.wav",
  "G": "sounds/frag_G2.wav",
  "H": "sounds/frag_H2.wav",  
}, {
  onload: () => {
    isLoaded.value = true;
    console.log("ドラム音源の読み込み完了！");
  }
}).toDestination(); // スピーカーに接続

// ■ 3. 再生機能
const playSequence = async () => {
  await Tone.start();
  
  if (!isLoaded.value) return;

  const now = Tone.now();
  let timeOffset = 0;

  noteBlocks.value.forEach((block) => {
    // 【重要】ここで音源をトリガーします
    // players.player("名前").start(いつ);
if (players.has(block.sound)) {
      const player = players.player(block.sound);
      
      // ★ここが重要：ファイルの長さ（秒）を取得
      // loadedでないとdurationは0になってしまうので注意
      const fileDuration = player.loaded ? player.buffer.duration : 0;

      // 2. 再生予約
      player.start(now + timeOffset);

      // 3. 見た目を光らせる予約 (Tone.Draw)
      Tone.Draw.schedule(() => {
        block.isPlaying = true;
      }, now + timeOffset);

      // 光るのを消す予約（ファイルの再生が終わるタイミングで消す）
      Tone.Draw.schedule(() => {
        block.isPlaying = false;
      }, now + timeOffset + fileDuration);
      
      // 4. 次の音の開始時間を更新
      // 「今までの時間 + このファイルの長さ」＝「次の開始時間」
      timeOffset += fileDuration; 
      
      // もし「間髪入れずに次に行くのが早すぎる」場合は、
      // ここに少し余白を足してもいいです
      // timeOffset += fileDuration + 0.1; // 0.1秒の余韻を入れる例
    }
  });
};

// ブロック追加
const addBlock = () => {
  noteBlocks.value.push({
    id: nextId++,
    sound: 'A',   // デフォルトはA

    color: '#555555'
  });
};

// ブロック削除
const removeBlock = (index) => {
  noteBlocks.value.splice(index, 1);
};

// 音色を変えた時に色も変えるヘルパー関数
const updateColor = (block) => {
  switch(block.sound) {
    // case 'A':  block.color = '##555555'; break;
    default:      block.color = '#555555';
  }
};
</script>

<template>
  <div class="container">
    <h1>DOPP Sequencer</h1>
    
    <div class="controls">
      <button 
        @click="playSequence" 
        class="btn-play" 
        :disabled="!isLoaded"
        :class="{ loading: !isLoaded }"
      >
        {{ isLoaded ? "▶ 再生" : "読み込み中..." }}
      </button>
      <button @click="addBlock" class="btn-add">＋ 追加</button>
    </div>

    <div class="sequencer-wrapper">
      <draggable 
        v-model="noteBlocks" 
        item-key="id" 
        class="block-list" 
        animation="200"
      >
        <template #item="{ element, index }">
          <div 
            class="music-block" 
            :style="{ backgroundColor: element.color }"
            :class="{ active: element.isPlaying }"
          >
            <span class="step-number">{{ index + 1 }}</span>

            <span class="sound-name">
              {{ element.sound.toUpperCase() }}
            </span>
            
            <select v-model="element.sound" @change="updateColor(element)" @click.stop>
              <option value="A">A</option>
              <option value="B">B</option>
              <option value="C">C</option>
              <option value="D">D</option>
              <option value="E">E</option>
              <option value="F">F</option>
              <option value="G">G</option>
              <option value="H">H</option>
            </select>
            
            <button class="btn-delete" @click.stop="removeBlock(index)">×</button>
          </div>
        </template>
      </draggable>
    </div>
  </div>
</template>

<style scoped>
/* 共通スタイルは維持 */
.container { max-width: 900px; margin: 0 auto; padding: 20px; background-color: #222; min-height: 100vh; color: white; font-family: sans-serif; }
.controls { margin-bottom: 20px; }
button { cursor: pointer; border: none; padding: 10px 20px; border-radius: 4px; font-weight: bold; margin-right: 10px; }
.btn-play { background-color: #4caf50; color: white; }
.btn-play:disabled { background-color: #555; }
.btn-add { background-color: #2e86de; color: white; }

.sequencer-wrapper { background-color: #333; padding: 20px; border-radius: 8px; min-height: 200px; }
.block-list { display: flex; flex-wrap: wrap; gap: 10px; }

/* ドラム用のブロックデザイン調整 */
.music-block { 
  width: 90px; 
  height: 90px; 
  border-radius: 4px; /* 少し角ばらせてパッドっぽく */
  display: flex; 
  flex-direction: column; 
  justify-content: center; 
  align-items: center; 
  cursor: grab; 
  position: relative; 
  box-shadow: 0 4px 6px rgba(0,0,0,0.3); 
  border: 2px solid rgba(255,255,255,0.1); /* 立体感を出す枠線 */
  transition: transform 0.1s, filter 0.1s, box-shadow 0.1s;
}

.music-block.active {
  /* 明るくして、枠線を太く光らせる */
  filter: brightness(1.3);
  box-shadow: 0 0 15px white;
  border-color: white;
  transform: scale(1.05); /* ほんの少し大きくする */
  z-index: 10; /* 隣のブロックより手前に表示 */
}

.step-number { position: absolute; top: 4px; left: 6px; font-size: 0.8rem; font-weight: bold; color: rgba(0,0,0,0.4); }
.sound-name { font-size: 1.0rem; font-weight: 900; color: #fff; text-shadow: 1px 1px 0 rgba(0,0,0,0.5); }
.duration { font-size: 0.7rem; color: rgba(255,255,255,0.8); margin-bottom: 4px;}
select { width: 80%; font-size: 0.7rem; background: rgba(225,225,225,0.2); color: '#555555'; border: none; border-radius: 2px; }
.btn-delete { position: absolute; top: 2px; right: 2px; background: rgba(0,0,0,0.2); color: white; width: 18px; height: 18px; padding: 0; font-size: 10px; line-height: 18px; border-radius: 50%; }
</style>