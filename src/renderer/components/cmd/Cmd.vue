<template>
    <div>
        <div class="controls">
            <v-btn color="success" @click="start"><v-icon>play_arrow</v-icon> Start</v-btn>
            <v-btn color="error" @click="kill"><v-icon>stop</v-icon> Stop</v-btn>
        </div>
        <div class="response">
            <section class="window">
                <header class="window-header">
                    <nav class="window-controls">
                        <span class="control-item control-minimize js-minimize">‒</span>
                        <span class="control-item control-maximize js-maximize">□</span>
                        <span class="control-item control-close js-close">˟</span>
                    </nav>
                </header>
                <main class="window-content">
                    <div class="window-cursor">
                        <div class="command">
                            <pre class="i-cursor-indicator">> </pre>
                            <input type="text" class="window-input js-prompt-input" v-model="fullCommand"/>
                        </div>
                        <div class="window-input stdout">{{ stdOut }}</div>
                    </div>
                </main>
            </section>
            <div class="prompt-shortcut i-prompt hidden js-open"></div>
        </div>
    </div>
</template>
<script>
    /* eslint-disable quotes */

    const { spawn, exec } = require('child_process')

export default {
        name: 'Cmd',
        props: {
            startDir: {
                type: String,
                default: function () {
                    return undefined
                }
            },
            command: {
                type: String,
                required: true
            },
            args: {
                type: Array,
                default: function () {
                    return []
                }
            },
            options: {
                type: Object,
                default: function () {
                    return {}
                }
            }
        },
        data: function () {
            return {
                stdOut: '',
                stdErr: '',
                bat: null
            }
        },
        computed: {
            fullCommand() {
                const args = this.args
                args.splice(0, 0, this.command)
                for (var key in this.options) {
                    if (typeof key === 'undefined') continue
                    if (!this.options.hasOwnProperty(key)) continue
                    const option = ((key.length > 1) ? '--' : '-') + key
                    const value = this.options[key]

                    args.push(option + ' ' + value)
                }
                console.log('Command: ' + args.join(' '))
                return args.join(' ')
            }
        },
        methods: {
            clearOut() {
                this.stdOut = ''
                this.stdErr = ''
            },
            start() {
                this.commandExec()
            },
            kill() {
                if (this.bat !== null) {
                    this.bat.kill('SIGHUP')
                }
            },
            commandExec() {
                this.kill()
                this.clearOut()
                this.bat = spawn('cmd.exe', ['/c', this.fullCommand], {
                    cwd: this.startDir,
                    env: {
                        PATH: process.env.PATH,
                        JAVA_HOME: process.env.JAVA_HOME
                    }
                })

                this.bat.stdout.on('data', (data) => {
                    // As said before, convert the Uint8Array to a readable string.
                    var str = String.fromCharCode.apply(null, data)
                    this.stdOut += str
                })

                // Handle error output
                this.bat.stderr.on('data', (data) => {
                    // As said before, convert the Uint8Array to a readable string.
                    var str = String.fromCharCode.apply(null, data)
                    this.stdErr += str
                })

                // Handle on exit event
                this.bat.on('exit', (code) => {
                    var preText = `Child exited with code ${code} : `

                    switch (code) {
                        case 0:
                            this.stdOut = 'Exited'
                            break
                        case 1:
                            this.stdErr += preText + 'The file already exists'
                            break
                        case 2:
                            this.stdErr += preText + "The file doesn't exists and now is created"
                            break
                        case 3:
                            this.stdErr += preText + 'An error ocurred while creating the file'
                            break
                    }
                })

                this.bat.on('close', () => {
                    console.log("Closed")
                })
            }
        },
        destroyed: function () {
            this.kill()
        }
    }
</script>
<style lang="scss">
    .window {
        width: 100%;
        height: 100vh;
        background-color: rgba(37,37,37,.85);
        border: 1px solid #c9c9c9;
        box-shadow: 4px 4px 0 rgba(0,0,0,.2);
        transition: all ease-in-out .3s;
    }
    .window.window--maximized {
        width: 100%;
        flex-grow: 1;
    }
    .window.window--minimized {
        height: 25px;
        width: 180px;
        .window-content { display: none; }
    }

    .window.window--destroyed {
        opacity: 0;
        width: 0;
        height: 0;
    }

    .window-header {
        background-color: #eee;
        height: 25px;
    }

    .window-controls {
        display: flex;
        height: 100%;
        justify-content: flex-end;
        align-items: center;
    }

    .window-controls .control-item {
        width: 25px;
        height: 25px;
        text-align: center;
        color: #333;
        transition: all ease-in .15s;
        cursor: pointer;

        &:hover { background-color: #ccc; }
    }

    .control-item.control-close {
        font-size: 20px;
        font-weight: 500;
        line-height: 28px;

        &:hover {
            background-color: #ec6060;
            color: #fff;
        }
    }
    .control-item.control-maximize { line-height: 19px; }
    .control-item.control-minimize { line-height: 22px; }

    .window-cursor {
        display: flex;
        width: 100%;
        flex-direction: column;
        margin: 5px;
        .i-cursor-indicator {
            display: inline-block;
            color: #fff;
            font-size: 1em;
            font-family: 'Consolas', monospace;
            margin: 0 2px 0 5px;
        }
        .i-cursor-underscore {
            width: 10px;
            height: 3px;
            background-color: #fff;
            align-self: flex-end;
            margin-right: 5px;
            animation: blink 1s steps(2, start) infinite;
        }
        .command {
            margin-bottom: 10px;
        }
        .stdout {
            width: 100%;
            overflow-y: scroll;
            height: 100vh;
        }
    }
    .window-input {
        background-color: inherit;
        border: none;
        outline: 0;
        color: transparent;
        text-shadow: 0 0 0 #fff;
        font-family: 'Consolas', monospace;
        flex: 1;

        &:focus {
            outline: none;
        }
    }

    .i-prompt {
        width: 62px;
        height: 62px;
        background: url("https://cdn4.iconfinder.com/data/icons/small-n-flat/24/terminal-48.png") no-repeat center;
        background-color: rgba(0,0,0,.35);
        border-radius: 10px;
        box-shadow: 0 3px 1px rgba(0,0,0,.25);
        cursor: pointer;
        transition: all ease-in-out .15s;
        &:hover {
            background-position: center 4px;
        }
    }
    .i-prompt.hidden {
        width: 0;
        height: 0;
        opacity: 0;
    }

    @keyframes blink {
        to { visibility: hidden; }
    }
</style>