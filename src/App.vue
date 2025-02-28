<template>
  <v-app class="overflow-hidden">
    <!--Main-->
    <v-app-bar app color="white" elevate-on-scroll :elevation="3">
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>Waterfall Project</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-menu bottom right>
        <template v-slot:activator="{ on }">
          <v-chip class="ma-2" label v-on="on">
            <v-icon left> mdi-cellphone </v-icon>
            <template v-if="devices[deviceChosen] != undefined">{{
              devices[deviceChosen].title
            }}</template>
            <template v-else>No Devices</template>
          </v-chip>
        </template>
        <v-list>
          <v-list-item
            v-for="(item, index) in devices"
            :key="index"
            @click="ChooseDevice(index)"
          >
            <template v-if="devices[index].status.match('device')">
              <v-icon left> mdi-cellphone </v-icon>
            </template>
            <template v-else>
              <v-icon left> mdi-lan-disconnect </v-icon>
            </template>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
      <v-btn icon color="blue" @click="GetDevices">
        <v-icon>mdi-cached</v-icon>
      </v-btn>
    </v-app-bar>
    <v-navigation-drawer v-model="drawer" fixed temporary>
      <v-list dense shaped>
        <v-list-item-group v-model="functionChosen" color="primary">
          <!--About Phone-->
          <v-list-item>
            <v-list-item-avatar>
              <v-icon>mdi-android</v-icon>
            </v-list-item-avatar>
            <v-list-item-content>
              <template v-if="devices.length > 0">
                {{ devices[deviceChosen].title }}
              </template>
            </v-list-item-content>
          </v-list-item>
          <!--Functions-->
          <v-list-item
            v-for="(item, i) in appFunctions"
            :key="i"
            :disabled="item.disable == true"
          >
            <v-list-item-icon>
              <v-icon>{{ item.icon }}</v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>{{ item.text }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <!--About Project-->
          <v-spacer />
          <v-list-item>
            <v-list-item-icon>
              <v-icon>mdi-information-outline</v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>About Project</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <!--End-->
        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>
    <v-main>
      <!--Manage your device-->
      <template v-if="devices.length > 0 && functionChosen != 4">
        <div>
          <template v-if="functionChosen == 0">
            <about-devices :deviceName="devices[deviceChosen].title" />
          </template>
          <template v-else-if="functionChosen == 1">
            <apk-manager :deviceName="devices[deviceChosen].title" />
          </template>
          <template v-else-if="functionChosen == 2">
            <file-manager :deviceName="devices[deviceChosen].title" />
          </template>
        </div>
      </template>
      <!--No device-->
      <template v-else-if="devices.length == 0 && functionChosen != 4">
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-container fill-height justify-center v-bind="attrs" v-on="on">
              <v-icon style="font-size: 500%"> mdi-progress-question </v-icon>
              <div class="text-h4 ml-3">No Devices!</div>
            </v-container>
          </template>
          <span>
            连接设备后，请点击<v-icon color="white" class="mx-2"
              >mdi-cached</v-icon
            >刷新设备列表。<br />
            若刷新后仍未出现您的设备，请确保打开了开发者模式：<br />
            1 进入「设置」中的「关于手机」，连续点击十次「版本号」。<br />
            2 进入「开发者选项」（ 请善用设置中的「搜索」功能
            ），确保其中的「USB调试」已开启。<br />
          </span>
        </v-tooltip>
      </template>
      <!--About Project-->
      <template v-else>
        <about-project />
      </template>
      <!--End-->
    </v-main>
    <!--If Fastboot or ADB not installed-->
    <template v-if="!adbInstalled || !fastbootInstalled">
      <v-row justify="center">
        <v-overlay :z-index="79" :opacity="0.75">
          <v-card elevation="6" color="red" class="pa-5">
            <v-list-item-title class="headline mr-16">
              出<b>错</b>了 !
            </v-list-item-title>
            <v-card-subtitle class="subtitle-1 mr-16">
              你的 Platform Tools <b>没有成功的配置</b> !
            </v-card-subtitle>
            <v-stepper v-model="e1" class="red elevation-0">
              <v-stepper-header class="elevation-0">
                <v-stepper-step :complete="e1 > 1" step="1" color="orange">
                  下载
                </v-stepper-step>
                <v-divider></v-divider>
                <v-stepper-step :complete="e1 > 2" step="2" color="orange">
                  解压
                </v-stepper-step>
                <v-divider></v-divider>
                <v-stepper-step :complete="e1 > 3" step="3" color="orange">
                  配置
                </v-stepper-step>
                <v-divider></v-divider>
                <v-stepper-step :complete="e1 > 4" step="4" color="orange">
                  完成
                </v-stepper-step>
              </v-stepper-header>

              <v-stepper-items class="elevation-0">
                <v-stepper-content step="1">
                  <v-card class="mb-12 elevation-0" color="red" height="200px">
                    下载 {{ GetOS() }} 平台的 Platform Tools :
                    <b
                      ><a
                        @click="
                          require('electron').shell.openExternal(
                            'https://dl.google.com/android/repository/'+platformTools[GetOS()]
                          )
                        "
                        class="white--text"
                        >{{ platformTools[GetOS()] }}</a
                      ></b
                    >.<br />
                    ( 您同时需要同意一份来自 Google 的条款及条件! )
                    <v-divider class="my-3" />
                    如果您有经验，您也可以自行配置 Path ，或者从其他地方下载
                    Platform Tools。
                  </v-card>
                  <v-spacer></v-spacer>
                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="e1 = 2"> 继续 </v-btn>
                  </v-card-actions>
                </v-stepper-content>

                <v-stepper-content step="2">
                  <v-card class="mb-12 elevation-0" color="red" height="200px">
                    解压压缩包中的 Platform-Tools 文件夹到一个您喜欢的地方,
                    请不要删除该文件夹 !
                  </v-card>
                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="e1 = 3"> 继续 </v-btn>
                  </v-card-actions>
                </v-stepper-content>

                <v-stepper-content step="3">
                  <v-card class="mb-12 elevation-0" color="red" height="200px">
                    请在对应的选择框中选择刚才解压的文件夹中的
                    <code>adb{{GetOS()=="Windows"?".exe":""}}</code> 或 <code>fastboot{{GetOS()=="Windows"?".exe":""}}</code> ,
                    选择完成后, 点击继续.
                    <v-file-input
                      class="mt-3"
                      label="选择 adb"
                      v-model="selectAdb"
                    ></v-file-input>
                    <v-file-input
                      label="选择 fastboot"
                      v-model="selectFastboot"
                    ></v-file-input>
                  </v-card>
                  <template
                    v-if="
                      this.selectFastboot.path != undefined &&
                      this.selectAdb.path != undefined
                    "
                  >
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn
                        text
                        @click="
                          GetUserInputPath();
                          e1 = 4;
                        "
                      >
                        继续
                      </v-btn>
                    </v-card-actions>
                  </template>
                </v-stepper-content>
                <v-stepper-content step="4">
                  配置完成！请刷新以应用更改！
                </v-stepper-content>
              </v-stepper-items>
            </v-stepper>
          </v-card>
        </v-overlay>
      </v-row>
    </template>
  </v-app>
</template>

<script>
const execSync = require("child_process").execSync;
const exec = require("child_process").exec;
const platform = require('process').platform; 
import AboutDevices from "./components/AboutDevices.vue";
import AboutProject from "./components/AboutProject.vue";
import ApkManager from "./components/APKManager.vue";
import FileManager from "./components/FileManager.vue";

export default {
  name: "App",

  components: {
    AboutDevices,
    AboutProject,
    ApkManager,
    FileManager,
  },

  data: function () {
    return {
      adb: null,
      fastboot: null,
      appLoading: true,
      devices: [],
      deviceChosen: "0",
      functionChosen: 0,
      appFunctions: [
        { text: "APK Manager", icon: "mdi-application" },
        { text: "Files Manager", icon: "mdi-file", disable: true  },
        { text: "Fastboot", icon: "mdi-android-debug-bridge", disable: true },
      ],
      adbInstalled: true,
      fastbootInstalled: true,
      userInputFilePath: [],
      drawer: false,
      e1: 1,
      platformTools: {
        "Windows": "platform-tools-latest-windows.zip",
        "MacOS": "platform-tools-latest-darwin.zip",
        "Linux": "platform-tools-latest-linux.zip",
      },
      selectAdb: [],
      selectFastboot: [],
      configPathWarning: false,
    };
  },

  created: function () {
    //Config Automatically If No Config
    if (
      localStorage.getItem("ADBPath") == null ||
      localStorage.getItem("FastbootPath") == null
    ) {
      localStorage.setItem("ADBPath", "adb");
      localStorage.setItem("FastbootPath", "fastboot");
    }
    //Set Path From Config
    this.adb = localStorage.getItem("ADBPath");
    this.fastboot = localStorage.getItem("FastbootPath");
    //Check
    this.CheckADB();
    this.GetDevices();
    this.appLoading = false;
  },

  watch: {
    group() {
      this.drawer = false;
    },
  },

  methods: {
    CloseWindow: function () {
      const { ipcRenderer } = require("electron");
      ipcRenderer.send("app:quit");
    },
    GetDevices: function () {
      this.devices = [];
      execSync(`${this.adb} devices`)
        .toString()
        .split("\n")
        .forEach((item, index) => {
          if (!item.match("List of devices attached") && item.length > 1) {
            this.devices.push({
              title: item.split("	")[0],
              status: item.split("	")[1],
              index: index,
            });
          }
        });
      //error:"device undefind" if using ForEach
      return;
    },
    ChooseDevice: function (index) {
      this.deviceChosen = index;
      return;
    },
    CheckADB: function () {
      exec(`${this.adb} --version`, (error) => {
        if (error) {
          this.adbInstalled = false;
          return;
        }
        this.adbInstalled = true;
        return;
      });
      exec(`${this.adb} --version`, (error) => {
        if (error) {
          this.fastbootInstalled = false;
          return;
        }
        this.fastbootInstalled = true;
        return;
      });
      return;
    },
    GetUserInputPath: function () {
      localStorage.setItem("ADBPath", `"` + this.selectAdb.path + `"`);
      localStorage.setItem(
        "FastbootPath",
        `"` + this.selectFastboot.path + `"`
      );
    },
    GetOS: function () {
      switch(platform) { 
        case 'darwin':  
          return "MacOS"  
        case 'win32':
          return 'Windows'
        case 'linux':
          return 'Linux'
        default:  
            return platform
      }
    }
  },
};
</script>

<style scoped>
</style>