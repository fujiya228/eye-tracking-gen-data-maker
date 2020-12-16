<template>
  <v-app>
    <v-navigation-drawer v-model="drawer" app permanent>
      <v-list>
        <v-list-item link @click="selectEditScreen('file')">
          <v-list-item-icon>
            <v-icon> mdi-file </v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>File / Settings</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item link @click="generateFlowData()">
          <v-list-item-icon>
            <v-icon>mdi-plus-box</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title> Generate </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item link @click="reverseFlowData()">
          <v-list-item-icon>
            <v-icon>mdi-swap-vertical</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>Reverse</v-list-item-title>
          </v-list-item-content>
        </v-list-item>

        <v-divider></v-divider>

        <v-list-item link @click="addFlowData()">
          <v-list-item-icon>
            <v-icon>mdi-plus</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title> Add </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <draggable v-model="flowData">
          <v-list-item
            v-for="(item, index) in flowData"
            :key="item.id"
            @click="selectEditScreen('flow', item)"
          >
            <v-list-item-content>
              <v-list-item-content
                >Target: {{ item.targetText }}
                {{ item.textData.slice(-1)[0].size }}-{{
                  item.textData.slice(-1)[0].alpha
                }}</v-list-item-content
              >
            </v-list-item-content>
            <v-list-item-icon>
              <v-btn icon right @click.stop="deleteFlowData(index)">
                <v-icon>mdi-delete</v-icon>
              </v-btn>
            </v-list-item-icon>
          </v-list-item>
        </draggable>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-container fluid>
        <v-row>
          <template v-if="selectedScreen.type === 'file'">
            <v-col cols="12">
              <v-card class="mx-auto" max-width="1200">
                <v-card-text>
                  <p class="display-1 text--primary">
                    {{ fileName !== "" ? fileName : "No file name" }}
                  </p>
                  <v-text-field
                    v-model="fileName"
                    label="File Name"
                    placeholder="File Name"
                    outlined
                    hide-details="auto"
                  ></v-text-field>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <input type="file" id="load" @change="load()" hidden />
                  <v-btn tile @click="clickLoad()"
                    ><v-icon left>mdi-upload</v-icon>Load</v-btn
                  >
                  <v-btn tile @click="download()"
                    ><v-icon left> mdi-download </v-icon>Download</v-btn
                  >
                </v-card-actions>
              </v-card>
            </v-col>
            <v-col cols="12">
              <v-card class="mx-auto" max-width="1200">
                <v-card-text>
                  <p class="display-1 text--primary">Global Settings</p>
                  <v-row>
                    <v-col
                      cols="6"
                      v-for="setting in globalSettings"
                      :key="setting.name"
                    >
                      <v-text-field
                        v-model.number="setting.value"
                        step="0.05"
                        type="number"
                        :label="setting.name"
                        :placeholder="setting.name"
                        outlined
                        hide-details="auto"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <p class="subtitle-2">Size List</p>
                  <v-row>
                    <v-col
                      v-for="(size, index) in sizeList"
                      :key="index"
                      cols="4"
                    >
                      <v-text-field
                        v-model.number="sizeList[index]"
                        type="number"
                        :label="index.toString()"
                        outlined
                        hide-details="auto"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <p class="subtitle-2">Alpha List</p>
                  <v-row>
                    <v-col
                      v-for="(alpha, index) in alphaList"
                      :key="index"
                      cols="4"
                    >
                      <v-text-field
                        v-model.number="alphaList[index]"
                        type="number"
                        :label="index.toString()"
                        step="0.05"
                        outlined
                        hide-details="auto"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <p class="subtitle-2">Target Character List</p>
                  <v-row>
                    <v-col
                      v-for="(target, index) in targetCharacterList"
                      :key="index"
                      cols="1.5"
                    >
                      <v-text-field
                        v-model="targetCharacterList[index]"
                        type="text"
                        :label="index.toString()"
                        outlined
                        hide-details="auto"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <p class="subtitle-2">Flow Range</p>
                  <v-row>
                    <v-col cols="6">
                      <v-text-field
                        v-model.number="flowMin"
                        type="number"
                        label="Min"
                        outlined
                        hide-details="auto"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="6">
                      <v-text-field
                        v-model.number="flowMax"
                        type="number"
                        label="Max"
                        outlined
                        hide-details="auto"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-card-text>
              </v-card>
            </v-col>
          </template>
          <template v-if="selectedScreen.type === 'flow' && selectedFlow">
            <v-col cols="12">
              <v-card class="mx-auto" max-width="1200">
                <v-card-text>
                  <p class="subtitle-1 text--primary">Target Text</p>
                  <v-text-field
                    v-model="selectedFlow.targetText"
                    label="Target Text"
                  ></v-text-field>
                </v-card-text>
              </v-card>
            </v-col>
            <v-col cols="12">
              <v-card class="mx-auto" max-width="1200">
                <v-card-text>
                  <p class="subtitle-1 text--primary">Text Data</p>
                </v-card-text>
                <v-list>
                  <v-list-item link @click="addTextData(selectedFlow)">
                    <v-list-item-content>
                      <v-list-item-title center> Add </v-list-item-title>
                    </v-list-item-content>
                    <v-list-item-icon>
                      <v-icon>mdi-plus</v-icon>
                    </v-list-item-icon>
                  </v-list-item>
                  <draggable v-model="selectedFlow.textData">
                    <v-list-item
                      v-for="(item, index) in selectedFlow.textData"
                      :key="item.id"
                    >
                      <v-list-item-content>
                        <v-col cols="1">
                          <v-icon>mdi-drag-horizontal-variant</v-icon>
                        </v-col>
                        <v-col cols="1">
                          <v-text-field
                            label="text"
                            placeholder="text"
                            outlined
                            hide-details="auto"
                            v-model="item.text"
                          ></v-text-field>
                        </v-col>
                        <v-col
                          v-for="param in textDataParams"
                          :key="param.name"
                          :cols="param.col"
                        >
                          <v-text-field
                            v-model.number="item[param.name]"
                            type="number"
                            :label="param.name"
                            :placeholder="param.name"
                            :step="param.step"
                            outlined
                            hide-details="auto"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="1">
                          <v-btn icon @click="deleteTextData(index)"
                            ><v-icon>mdi-delete</v-icon></v-btn
                          >
                        </v-col>
                      </v-list-item-content>
                    </v-list-item>
                  </draggable>
                </v-list>
              </v-card>
            </v-col>
          </template>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import draggable from "vuedraggable";
export default {
  components: {
    draggable,
  },
  data: () => ({
    drawer: null,
    fileName: "",
    selectedScreen: {
      type: "file",
      id: -1,
    },
    textDataParams: [
      { name: "size", col: 1, step: 1 },
      { name: "alpha", col: 1, step: 1 },
      { name: "deg", col: 2, step: 5 },
      { name: "speed", col: 2, step: 0.05 },
      { name: "interval", col: 2, step: 0.05 },
    ],
    selectedFlow: null,
    flowCount: 3,
    globalSettings: {
      interval: {
        name: "Interval",
        value: 1.5,
      },
      speed: {
        name: "Speed",
        value: 3,
      },
      genRadius: {
        name: "Generate Radius",
        value: 0.5,
      },
      delRadius: {
        name: "Delete Radius",
        value: 2.5,
      },
    },
    SizeAlpha: [
      [0, 0],
      [0, 1],
      [0, 2],
      [1, 0],
      [1, 1],
      [1, 2],
      [2, 0],
      [2, 1],
      [2, 2],
    ],
    sizeList: [16, 32, 48],
    alphaList: [0.3, 0.6, 1],
    targetCharacterList: ["A", "B", "D", "E", "F", "G", "H", "J", "M"],
    flowMax: 8,
    flowMin: 4,
    flowData: [],
  }),
  methods: {
    selectEditScreen(type, flow = null) {
      this.selectedFlow = flow;
      this.selectedScreen.type = type;
      if (flow) {
        this.selectedScreen.id = flow.id;
      }
    },
    addFlowData(text = "A") {
      this.flowData.push({
        id: this.flowCount++,
        targetText: text,
        textCount: 0,
        textData: [],
      });
    },
    deleteFlowData(index) {
      this.flowData.splice(index, 1);
      this.selectedFlow = null;
      this.selectedScreen.type = "file";
    },
    addTextData(flow, text = flow.targetText, size = 0, alpha = 0, deg = 0) {
      flow.textData.push({
        id: flow.textCount++,
        text: text,
        size: size,
        alpha: alpha,
        rad: 0,
        deg: deg,
        x: 0,
        y: 0,
        speed: this.globalSettings.speed.value,
        interval: this.globalSettings.interval.value,
      });
    },
    deleteTextData(index) {
      this.selectedFlow.textData.splice(index, 1);
    },
    generateFlowData() {
      this.selectedScreen.type = "file";
      this.selectedScreen.id = -1;
      this.selectedFlow = null;
      // fileName作成
      let dateList = new Date(Date.now()).toLocaleString().split(/\/| |:/);
      this.fileName = "sample";
      dateList.forEach((item) => {
        this.fileName += "-" + item;
      });
      // this.fileName += ".json";
      // データをランダム生成
      this.flowData = [];
      this.flowCount = 0;
      this.targetCharacterList.sort(); // Generate時にソート
      const flowNum = 9; // 実験条件そろえるので9で固定に
      const randomTargets = this.shuffle([...this.targetCharacterList]); // targetの文字の順序をランダムに
      const randomSizeAlpha = this.shuffle([...this.SizeAlpha]); // targetの文字のサイズ,α値の組み合わせをランダムに
      for (let i = 0; i < flowNum; i++) {
        this.addFlowData(randomTargets[i]); // targetの文字をセット
        let textNum = this.getRandomInt(this.flowMin - 1, this.flowMax); // 文字の数は flowMin-1 以上 flowMax未満でランダムに（最後にtarget入れるので-1）
        // target以外の文字の配列を作成
        let nonTargetCharacterList = [];
        for (let i = 10; i < 36; i++)
          nonTargetCharacterList.push(i.toString(36).toUpperCase());
        // targetを除去
        nonTargetCharacterList.splice(parseInt(randomTargets[i], 36), 1);
        // target以外のフロー作成
        for (let j = 0; j < textNum; j++) {
          let text = this.getRandomChar(nonTargetCharacterList); // target以外の文字からランダムに
          let size = this.getRandomInt(0, 3);
          let alpha = this.getRandomInt(0, 3);
          let deg = this.getRandomInt(0, 360);
          this.addTextData(this.flowData[i], text, size, alpha, deg);
        }
        // 最後にtargetの文字をセット
        this.addTextData(
          this.flowData[i],
          randomTargets[i],
          randomSizeAlpha[i][0],
          randomSizeAlpha[i][1],
          this.getRandomInt(0, 360)
        );
      }
    },
    reverseFlowData() {
      this.flowData = this.flowData.reverse();
    },
    getRandomInt(min, max) {
      min = Math.ceil(min);
      max = Math.floor(max);
      return Math.floor(Math.random() * (max - min) + min);
    },
    getRandomChar(list = null) {
      if (list == null)
        return this.getRandomInt(0, 36).toString(36).toUpperCase();
      return list[this.getRandomInt(0, list.length)];
    },
    deepCopy(data) {
      return JSON.parse(JSON.stringify(data));
    },
    shuffle(array) {
      for (let i = array.length - 1; i >= 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    },
    getArrayItemByIndex(array, index) {
      if (index < 0) index = array.length + index;
      return array[index];
    },
    clickLoad() {
      document.getElementById("load").click();
    },
    load() {
      let data;
      let result = event.target.files[0];
      let reader = new FileReader();
      if (result === null) return;
      this.fileName = result.name.split(".")[0];
      reader.readAsText(result);
      reader.addEventListener("load", () => {
        data = JSON.parse(reader.result);
        this.initData(data);
      });
    },
    initData(data) {
      this.targetCharacterList = []; // targetのテキストを更新
      data.flowData.forEach((flowItem, flowIndex) => {
        flowItem.id = flowIndex;
        flowItem.textCount = flowItem.textData.length;
        flowItem.textData.forEach((textItem, textIndex) => {
          textItem.id = textIndex;
        });
        this.targetCharacterList.push(flowItem.textData.slice(-1)[0].text); // targetのテキストを更新
      });
      this.targetCharacterList.sort(); // targetのテキストを更新
      this.selectedScreen.type = "file";
      this.selectedScreen.id = -1;
      this.flowCount = data.flowNum;
      this.globalSettings.interval.value = data.interval;
      this.globalSettings.speed.value = data.speed;
      this.globalSettings.genRadius.value = data.genRadius;
      this.globalSettings.delRadius.value = data.delRadius;
      this.sizeList = data.sizeList;
      this.alphaList = data.alphaList;
      this.flowData = data.flowData;
    },
    download() {
      let data = {
        flowNum: this.flowData.length,
        interval: this.globalSettings.interval.value,
        speed: this.globalSettings.speed.value,
        genRadius: this.globalSettings.genRadius.value,
        delRadius: this.globalSettings.delRadius.value,
        sizeList: this.sizeList,
        alphaList: this.alphaList,
        flowData: this.deepCopy(this.flowData),
      };
      data.flowData.forEach((flowItem) => {
        delete flowItem.id;
        delete flowItem.textCount;
        flowItem.textData.forEach((textItem) => {
          delete textItem.id;
          textItem.rad = (textItem.deg * Math.PI) / 180;
          textItem.x =
            this.globalSettings.genRadius.value * Math.cos(textItem.rad);
          textItem.y =
            this.globalSettings.genRadius.value * Math.sin(textItem.rad);
        });
      });
      var obj = JSON.stringify(data);
      let link = document.createElement("a");
      link.href = "data:text/json," + encodeURIComponent(obj);
      link.download = this.fileName + ".json";
      link.click();
      link.remove();
    },
  },
  created() {},
};
</script>