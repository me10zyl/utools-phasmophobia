<template>
  <div>
    <el-container>
      <el-main>
        <div>
          <span style='font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;line-height:1.5 ;font-size: 18px'>选择证据</span>
          <el-button style="float: right" circle type="danger" @click="reset">重置</el-button>
        </div>
        <div v-for="(evidences,i) in selectedEvidenceList">
          <el-row style="margin-top: 20px">
            <span style="padding-right: 15px">{{i}}. </span>
            <el-button round v-for="evidence in evidences" @click="clickEvidence(evidence, evidences)"
                       v-bind:type="evidence.select ? 'primary' : ''">{{evidence.name}}
            </el-button>
          </el-row>
        </div>
        <el-row v-for="ghostTypeRow in filteredGhostTypeList" style="margin-top: 20px" :gutter="12">
          <el-col v-for="ghostType in ghostTypeRow"  :span="8">
            <el-card shadow="always">
              <div style='color:green'>
                {{ghostType.name}}
              </div>
               <div>
                {{ghostType.desc}}
              </div>
              <div>
                {{ghostType.weakness}}
              </div>
              <div>
                {{ghostType.feature}}
              </div>
              <div v-html="ghostType.evidence.join('、')">
              </div>
            </el-card>
          </el-col>
        </el-row>
      </el-main>
    </el-container>
  </div>
</template>

<script>
  export default {
    name: 'HelloWorld',
    computed: {
      filteredGhostTypeList: function (newVal, oldVal) {
        //选中鬼魂判断
        let filteredList = [];
        this.selectedEvidenceList.forEach(e => {
          let filtered = e.filter(ee => ee.select === true)[0];
          if (filtered) {
            filteredList.push(filtered.name);
          }
        })
        //console.log(filteredList)
        if (filteredList.length == 0) {
          return [];
        }
        let ret = [];
        let filt = this.ghostTypes.filter(ghostType => {
          let evidenceList = ghostType.evidence;
          let allContains = true;
          for (let i in filteredList) {
            if (evidenceList.indexOf(filteredList[i]) <= -1) {
              //console.log(ghostType.name + ' not contains', filteredList[i])
              allContains = false;
              break;
            }
          }
          return allContains;
        });
        ret = this.deepClone(filt);
        for (let i in ret) {
          let evidence = ret[i].evidence;
          for (let j in evidence) {
            if (filteredList.indexOf(evidence[j]) >= 0) {
              evidence[j] = "<font color='#FF0000'>" + evidence[j] + "</font>"
            }
          }
          ret[i].evidence.sort()
        }
        let datas = [];
        for(let i in ret){
          if(i % 3 == 0){
             let newarr = [];
             datas.push(newarr)
          }
          datas[datas.length - 1].push(ret[i]);
        }
        console.log(datas)
        return datas;
      }
    },
    methods : {
      reset : function(){
        this.selectedEvidenceList.forEach(e=>e.forEach(ee=>ee.select=false));
      }
    },
    data() {
      return {
        deepClone(data) {
         return  JSON.parse(JSON.stringify(data));
        },
        clickEvidence: function (evidence, evidences) {
          evidence.select = !evidence.select;
          evidences.filter(e => e !== evidence).forEach(e => e.select = false)
        },
        selectedEvidenceList: [],
        evidences: ['通灵盒', '指纹', '笔迹', '寒温', 'EMF5', '灵球'],
        ghostTypes: [
          {
            name: '魂魄',
            feature: '无',
            weakness: '弱点：在魂魄附近点燃圣木会阻止它的攻击, 阻止时间为三分钟。',
            evidence: ["通灵盒", "指纹", "笔迹"],
            desc : '魂魄是最常见的鬼魂，但它仍不容小觑。它通常会在死者死因不明的区域出现。'
          },
          {
            name: '死灵',
            feature: '特征：不接触地面, 所以他没有足迹',
            weakness: '弱点：碰触到盐在十秒钟内会变得非常活跃。',
            evidence: ["指纹", "寒温", "通灵盒"],
            desc : '死灵是最危险的鬼魂之一。它也是唯一已知的具有飞行能力的鬼魂，有时他还能穿墙。'
          },
          {
            name: '幻影',
            feature: '特征：看到祂会大幅降低理智。',
            weakness: '弱点：拍照祂会暂时消失(照片拍不到他)。',
            evidence: ["EMF5", "灵球", "寒温"],
            desc : '幻影是种能附身的鬼魂，通常可用通灵板召唤。它也会让周围的人感到恐惧。'
          },
          {
            name: '骚灵',
            feature: '特征：祂会一次大量丢物体。',
            weakness: '弱点：空无一物房间毫无用处。',
            evidence: ["通灵盒", "指纹", "灵球"],
            desc : '鬼魂中最著名的一种鬼，也称为吵闹鬼。他可以通过操纵周围的物体来散播恐惧。'
          },
          {
            name: '女妖',
            feature: '特征：女妖一次只锁定一个人',
            weakness: '弱点：女妖害怕十字架(范围5尺)',
            evidence: ["EMF5", "指纹", "寒温"],
            desc : '女妖是天生的猎人，它会攻击任何东西。它只会跟踪一个目标直至其死亡。'
          },
          {
            name: '巨灵',
            feature: '特征：如果目标在远处，会更快速移动。',
            weakness: '弱点：关闭电源他移动速度会变慢, 防止使用其技能',
            evidence: ["通灵盒", "灵球", "EMF5"],
            desc : '巨灵是一种地域意识很强的鬼魂，受到威胁时会发动攻击。它能够以相当快的速度移动。'
          },
          {
            name: '梦魇',
            feature: '特征：黑暗中更有可能攻击你。',
            weakness: '弱点：开灯会降低牠攻击慾望。',
            evidence: ["通灵盒", "灵球", "寒温"],
            desc : '梦魇是噩梦之源，他是黑暗中最强大的梦魇。'
          },
          {
            name: '亡魂',
            feature: '特征：狩猎时会高速移动。',
            weakness: '弱点：找不到猎物移动缓慢。',
            evidence: ["EMF5", "指纹", "笔迹"],
            desc : '亡魂移动缓慢，但很狂暴，他会肆意攻击。传言道他狩猎时会高速移动。'
          },
          {
            name: '暗影',
            feature: '特征：害羞，不敢出来，很难找到。',
            weakness: '弱点：如果附近很多人，就不会进入狩猎。',
            evidence: ["EMF5", "灵球", "笔迹"],
            desc : '暗影是个害羞鬼。证据表明，只要人够多他就不敢活动。'
          },
          {
            name: '恶魔',
            feature: '特征：更有进攻性(很容易进入狩猎)',
            weakness: '弱点：通灵板使用成功不会降低理智。',
            evidence: ["通灵盒", "笔迹", "寒温"],
            desc : '恶魔会是你最不想遇到的鬼之一。众所周知，它会无缘无故地攻击你。'
          },
          {
            name: '幽灵',
            feature: '特征：黑暗中理智失去更快。',
            weakness: '弱点：在他所在的房间点燃圣木会让他离开房间一阵子。',
            evidence: ["灵球", "笔迹", "寒温"],
            desc : '幽灵是回到现实世界的复仇之魂。'
          },
          {
            name: '赤鬼',
            feature: '特征：如果有人在附近祂会更活跃，他会高速移动物品。',
            weakness: '弱点：玩家反应更加剧烈，会引起祂兴趣。(玩家开门关门、开灯关灯、丢东西制造噪音、走动)',
            evidence: ["EMF5", "通灵盒", "笔迹"],
            desc : '赤鬼是恶魔的亲戚，拥有超强的力量。传言道他们会在猎物周围变得更加活跃。'
          }
        ]
      }
    },
    mounted() {
      for (let i = 0; i < 3; i++) {
        let data = [];
        this.evidences.forEach(e => {
          data.push({
            name: e,
            select: false
          })
        })
        this.selectedEvidenceList.push(data);
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }
</style>
