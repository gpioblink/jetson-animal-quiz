<template>
  <v-container>
    <v-layout
      wrap
    >
      <v-row>
        <v-flex xs12>
          <v-card>
            <v-card-title>
              Question
            </v-card-title>
            <v-card-text>
            <v-container>
              <v-row xs8 justify='center'>
                <v-col xs8 md8>
                <v-alert
                  outlined
                  color="black"
                  style="white-space:pre-wrap"
                >
                  <div class="title">Riddle</div>
                  <div>{{problems[selectedQuiz].sentence}}</div>
                </v-alert>
                </v-col>
              </v-row>
              </v-container>
            </v-card-text>
            <v-card-actions>
              <v-layout justify-center>
                <v-btn v-if="!isJuding" text icon>Please Show me animal...</v-btn>
              </v-layout>
            </v-card-actions>
            <v-layout justify-center>
            <v-flex xs4>
              <v-img v-if="isJuding && isCorrect" :src="judgeSrc.right"></v-img>
              <v-img v-if="isJuding && !(isCorrect)" :src="judgeSrc.wrong"></v-img>
              <v-img v-if="!isJuding" :src="judgeSrc.none"></v-img>
            </v-flex>
            </v-layout>
          </v-card>
        </v-flex>
      </v-row>
    </v-layout>
  </v-container>
</template>

<script>
import mqtt from 'mqtt';
import { Howl } from 'howler';
import { isContext } from 'vm';
export default {
  data: () => ({
    judgeSrc: {
      right: require('../assets/mark_maru.png'),
      wrong: require('../assets/mark_batsu.png'),
      none: require('../assets/mark_none.png')
    },
    problems: {
      cat: {
        sentence: 'Be careful with this type of pet\nAs some may scratch you with their claws\nSome of them always stay inside\nWhile some like to roam outdoors',
        sound: require('../assets/sound/cat.mp3')
      },
      /*fox: {
        sentence: 'I have three heads. Cut off one, I become stronger. Cut off two, I become ten. What am I?',
        sound: require('../assets/sound/fox.mp3')
      },*/
      chicken: {
        sentence: 'You might be called this animal\nIf someone thinks that you’re afraid\nThis is something that you might eat\nAs well as its eggs that it laid',
        sound: require('../assets/sound/chicken.mp3')
      },
      cow: {
        sentence: 'They live in a field\nMilk is what they make\nThey help give us leather\nAnd a juicy steak',
        sound: require('../assets/sound/cow.wav')
      },
      pig: {
        sentence: 'This is a type of animal\nWhich has a curly tail\nA female is called a sow\nAnd a boar is the male',
        sound: require('../assets/sound/pig.wav')
      }
    },
    audio: {
      tempo: null,
      beat: null
    },
    isJuding: false,
    selectedQuiz: "cat",
    isCorrect: false
  }),
  mounted: function() {
    const client = mqtt.connect('ws://192.168.55.1:9090');
    this.selectedQuiz = '';
    const keys = Object.keys(this.problems);
    this.selectedQuiz = keys[Math.floor(Math.random() * keys.length)];
    client.publish('orz', 'hello!');
    client.on('connect', function(){
        console.log('subscriber.connected.');
    });

    client.subscribe('orz', function(err, granted){
        console.log('subscriber.subscribed.');
    });
    client.on('message', (topic, message) => {
      if(this.isJuding == false) {
        this.isCorrect = (this.selectedQuiz == message.toString())? true: false;
        this.isJuding = true;
        console.log('subscriber.on.message', 'topic:', topic, 'message:', message.toString());
        console.log(this.selectedQuiz, message.toString() , this.selectedQuiz == message.toString(), this.isCorrect);
        if(!(message.toString() in keys)){
          const soundOption = {
              src: [this.problems[message.toString()].sound],
              autoplay: true,
              loop: false,
              volume: 0.5,
              onend: () => { // 再生完了時のendイベント
                this.isJuding = false;
                if(this.isCorrect) this.selectedQuiz = keys[Math.floor(Math.random() * keys.length)];
              }
          }
          const sound = new Howl(soundOption);
        }
      }
    });
}
};
</script>