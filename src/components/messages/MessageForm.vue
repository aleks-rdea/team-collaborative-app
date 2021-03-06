<template>
    <div class="messages__form">
        <div class="ui inverted form">
            <div class="two fields">
                <div class="field">
                    <textarea 
                        name="message" 
                        id="message" 
                        v-model.trim="message" 
                        rows="3" 
                        placeholder="Message"
                    ></textarea>
                </div>
                <div class="field">
                    <button class="ui green button" @click.prevent="sendMessage">Send</button>
                    <button 
                        class="ui labeled icon button" 
                        @click.prevent="openFileModal"
                        :class="{'disabled' : uploadState == 'uploading'}"
                    ><i class="cloud upload icon"></i>File</button>
                </div>
            </div>
        </div>
        <div 
            class="ui small orange inverted progress" 
            data-total="100" 
            id="uploadedFile"
            v-if="uploadState != null"
        >
            <div class="bar">
                <div class="progress"></div>
            </div>
            <div class="label">{{ uploadLabel }}</div>
        </div>
        <file-modal ref="file_modal"></file-modal>
    </div>
</template>

<script>

    import {mapGetters} from 'vuex'
    import FileModal from './FileModal'
    import uuidV4 from 'uuid/V4'

    export default {
        name: 'message-form',
        components: {FileModal},
        data() {
            return {
                message: '',
                errors: [],
                uploadTask: null,
                uploadState: null,
                storageRef: firebase.storage().ref(),
            }
        },
        computed: {
            ...mapGetters(['currentChannel', 'currentUser', 'isPrivate']),
            uploadLabel() {
                switch(this.uploadState) {
                    case 'uploading': return 'Sending in progress...'
                        break;
                    case 'error': return 'An error s\'has occurred'
                        break;
                    case 'done': return 'Send made'
                        break;
                    default: return ''
                }
            }
        },
        methods: {
            sendMessage() {
                if(this.currentChannel !==null) {
                    if(this.message.length > 0) {
                       this.$parent.getMessageRef().child(this.currentChannel.id).push().set(this.createMessage()).then(() => {

                        }).catch(error => {
                            this.errors.push(error.message)
                        })
                        this.message = "" 
                    }
                }
            },
            createMessage( fileUrl = null) {
                let message = {
                    timestamp: firebase.database.ServerValue.TIMESTAMP,
                    user: {
                        name: this.currentUser.displayName,
                        avatar: this.currentUser.photoURL,
                        id: this.currentUser.uid
                    }
                }
                if(fileUrl == null) {
                    message['content'] = this.message
                } else {
                    message['image'] = fileUrl
                }
                return message
            },
            uploadFile(file, metadata) {
                if(file === null) return false

                let pathToUpload = this.currentChannel.id
                let ref = this.$parent.getMessageRef()
                let filePath = this.getPath() + '/' + uuidV4() + '.jpg'

                this.uploadTask = this.storageRef.child(filePath).put(file, metadata)
                this.uploadState = "uploading"
                this.uploadTask.on('state_changed', snap => {
                    let percent = (snap.bytesTransferred / snap.totalBytes) * 100
                    $("#uploadedFile").progress("set percent", percent)
                }, error => {
                    this.errors.push(error.message)
                    this.uploadState = 'error'
                    this.uploadTask = null
                }, () => {
                    this.uploadState = 'done'
                    this.$refs.file_modal.resetForm()

                    let fileUrl = this.uploadTask.snapshot.downloadURL
                    this.sendFileMessage(fileUrl, ref, pathToUpload)
                })
            },
            sendFileMessage(fileUrl, ref, pathToUpload) {
                ref.child(pathToUpload).push().set(this.createMessage(fileUrl)).then(() => {
                    this.$nextTick(() => {
                        $("html, body").scrollTop($(document).height())
                    })
                }).catch(error => {
                    this.errors.push(error.message)
                })
            },
            openFileModal() {
                $("#fileModal").modal("show")
            },
            getPath() {
                if(this.isPrivate) {
                    return 'tchat/private/'+this.currentChannel.id
                } else {
                    return 'tchat/public'
                }
            }
        },
        beforeDestroy() {
            if(this.uploadTask !== null) {
                this.uploadTask.cancel()
                this.uploadTask = null
            }
        }
    }
</script>

<style scoped>
    .message__form {
        position: fixed;
        bottom: 0;
        background-color: #232323;
        padding: 10px;
        padding-top: 20px;
        height: 210px;
        left: 300px;
        right: 0; 
    }
    .message__form.big {
        height: 350px;
    }
    .shortcut {
        color: white;
    }
</style>


