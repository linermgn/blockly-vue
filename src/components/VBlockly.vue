<template>
    <div>
        <div class="blockly-div"></div>
    </div>
</template>

<script>
    import Vue from 'vue'
    import Blockly from 'blockly'

    // ignore xml element and block element
    Vue.config.ignoredElements.push('mutation')
    Vue.config.ignoredElements.push('arg')
    Vue.config.ignoredElements.push('xml')
    Vue.config.ignoredElements.push('block')
    Vue.config.ignoredElements.push('variables')
    Vue.config.ignoredElements.push('variable')
    Vue.config.ignoredElements.push('field')

    export default {
        name: 'VBlockly',
        props: {
            toolbox: {
                type: String
            },
            workspace: {
                type: String
            }
        },
        data() {
            return {
                currentToolbox: null,
                currentWorkspace: null,
                code: '',
                xml: '',
                resizeObserver: null
            }
        },
        mounted() {
            Promise.all([this.injectBlockly(), this.loadWorkspace()])
                .then(values => {
                    let workspace = values[1]
                    if (workspace) {
                        this.$el.appendChild(workspace)
                        Blockly.Xml.domToWorkspace(workspace, this.currentWorkspace)
                    }
                    this.currentWorkspace.addChangeListener(this.workspaceChanged)
                })
                .finally(() => {
                    this.observeResizes()
                });
        },
        methods: {
            fetchXML(xmlName) {
                if (!xmlName) {
                    return Promise.resolve(null)
                }
                let parser = new DOMParser()
                return fetch(xmlName)
                    .then(resp => resp.text())
                    .then(text => {
                        return parser.parseFromString(text, 'application/xml').firstChild // return xml
                    })
            },
            loadToolbox() {
                return this.fetchXML(this.toolbox)
            },
            loadWorkspace() {
                return this.fetchXML(this.workspace)
            },
            injectBlockly() {
                let blocklyDiv = this.$el.querySelector('.blockly-div')

                return this.loadToolbox()
                    .then((toolbox) => {
                        // base configuration
                        let options = {
                            scrollbars: true,
                            sounds: true,
                            oneBasedIndex: true,
                            grid: {spacing: 20, length: 1, colour: '#888', snap: true}
                        }

                        // add a toolbox if we have one
                        if (toolbox) {
                            this.$el.appendChild(toolbox)
                            options.toolbox = toolbox
                            this.currentToolbox = toolbox
                        }

                        // inject Blockly
                        this.currentWorkspace = Blockly.inject(blocklyDiv, options)

                        return Promise.resolve(this.currentWorkspace)
                    })
            },
            resize() {
                // if we don't have a workspace we're done
                if (!this.currentWorkspace) {
                    return
                }
                // resize the  workspace
                console.log('resize')
                Blockly.svgResize(this.currentWorkspace)

                // Fix collapse bug
                let blocks = this.currentWorkspace.getTopBlocks()
                blocks.map(block => block.setCollapsed(true))
                blocks.map(block => block.setCollapsed(false))
            },
            observeResizes() {
                let observer = new ResizeObserver(this.resize)
                observer.observe(this.$el)
                this.resizeObserver = observer
            },
            workspaceChanged(event) {
                let name = event.type
                if (event.type === Blockly.Events.BLOCK_CHANGE) {
                    this.$emit(name, {
                        event,
                        workspace: this.currentWorkspace,
                        block: this.currentWorkspace.getBlockById(event.blockId)
                    })
                } else if (event.type === Blockly.Events.BLOCK_CREATE) {
                    this.$emit(name, {
                        event,
                        workspace: this.currentWorkspace,
                        block: this.currentWorkspace.getBlockById(event.blockId)
                    })
                } else {
                    // for all other events emit the workspace
                    this.$emit(name, {
                        event,
                        workspace: this.currentWorkspace
                    })
                }
                // if (this.stub) {
                //     this.code = Blockly.Python.workspaceToCode(this.currentWorkspace)
                // }
                let dom = Blockly.Xml.workspaceToDom(this.currentWorkspace)
                let xml = Blockly.Xml.domToPrettyText(dom)
                // let xml = new XMLSerializer().serializeToString(dom)
                this.xml = xml
            }
        }
    }
</script>


<style>
    /* Fix layout issues. This can't be scoped because edit  widgets are placed outside of the component */
    .blocklyWidgetDiv {
        /* Something like this....*/
        margin: -1.55px 0 0 2.5px;
    }

    /*  */
</style>
<style scoped>
    code:after, code:before, kbd:after, kbd:before {
        content: "";
        letter-spacing: initial;
    }

    /*.blockly-flex {*/
    /*    display: flex;*/
    /*    height: 100%;*/
    /*}*/
    .blockly-div {
        height: 100%;
        width: 100%;
    }

    /*.blockly-code {*/
    /*    margin:5px;*/
    /*    flex: 1;*/
    /*}*/
    /*.blockly-toolbox {*/
    /*    display: none;*/
    /*}*/
</style>