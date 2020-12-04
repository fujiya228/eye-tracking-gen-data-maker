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
                >Target: {{ item.targetText }}</v-list-item-content
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
                  <p class="subtitle-2">Size List</p>
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
    sizeList: [16, 32, 48],
    alphaList: [0.3, 0.6, 1],
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
      let flowNum = Math.floor(Math.random() * 7) + 3;
      for (let i = 0; i < flowNum; i++) {
        this.addFlowData(this.getRandomChar());
        let textNum = Math.floor(Math.random() * 7) + 3;
        for (let j = 0; j < textNum; j++) {
          let text =
            j === textNum - 1
              ? this.flowData[i].targetText
              : this.getRandomChar();
          let size = this.getRandomInt(3);
          let alpha = this.getRandomInt(3);
          let deg = this.getRandomInt(360);
          this.addTextData(this.flowData[i], text, size, alpha, deg);
        }
      }
    },
    reverseFlowData() {
      this.flowData = this.flowData.reverse();
    },
    getRandomInt(max) {
      return Math.floor(Math.random() * Math.floor(max));
    },
    getRandomChar() {
      return this.getRandomInt(36).toString(36).toUpperCase();
    },
    deepCopy(data) {
      return JSON.parse(JSON.stringify(data));
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
      data.flowData.forEach((flowItem, flowIndex) => {
        flowItem.id = flowIndex;
        flowItem.textCount = flowItem.textData.length;
        flowItem.textData.forEach((textItem, textIndex) => {
          textItem.id = textIndex;
        });
      });
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
};
</script>