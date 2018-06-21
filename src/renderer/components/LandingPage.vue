<template>
  <div id="wrapper">
    <v-app id="inspire">
      <div>
        <v-toolbar color="cyan" dark tabs>
          <v-toolbar-side-icon></v-toolbar-side-icon>
          <v-toolbar-title>Title</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-tabs
                  slot="extension"
                  v-model="tab"
                  color="cyan"
                  grow
          >
            <v-tabs-slider color="yellow"></v-tabs-slider>
            <v-tab v-for="tab in tabs">
              {{ tab.name }}
            </v-tab>
          </v-tabs>
        </v-toolbar>
        <v-tabs-items v-model="tab">
          <v-tab-item v-for="tab in tabs">
            <v-card flat style="padding: 10px 20px 10px ;">
              <Cmd
                      :command="tab.command"
                      :startDir="tab.startDir"
                      :args="tab.args"
                      :options="tab.options"
              />
            </v-card>
          </v-tab-item>
        </v-tabs-items>
      </div>
    </v-app>
  </div>
</template>

<script>
  import Cmd from './cmd/Cmd'
  import { Tabs, Tab } from 'vue-tabs-component'

  export default {
      name: 'landing-page',
      components: { Cmd, Tabs, Tab },
      data: function () {
          return {
              tab: 0,
              tabs: [
                  {
                      name: 'Apache Server',
                      command: 'httpd.exe',
                      startDir: 'C:\\dev\\servers\\apache\\bin',
                      args: [],
                      options: {}
                  },
                  {
                      name: 'JBoos Deploy',
                      command: 'standalone.bat',
                      startDir: 'C:\\dev\\servers\\jboss-eap-6.4\\bin',
                      args: [],
                      options: {}
                  },
                  {
                      name: 'Grunt',
                      command: 'grunt',
                      startDir: 'C:\\dev\\repository\\frontend-html5',
                      args: [],
                      options: {}
                  },
                  {
                      name: 'Test',
                      command: 'ping',
                      startDir: '',
                      args: [
                          '127.0.0.1'
                      ],
                      options: {
                          n: 15
                      }
                  }
              ]
          }
      },
      methods: {
          grunt(args = [], options = []) {
              this.tabs[3].args = args
              this.tabs[3].options = options
          }
      }
  }
</script>

<style>
  .controls {
    padding: 10px 0;
    text-align: right;
  }
</style>
