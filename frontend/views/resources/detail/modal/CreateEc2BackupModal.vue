<template>
    <v-layout row justify-center>
        <v-dialog v-model="isOpen" persistent max-width="400">
            <v-card>
                <v-card-title class="headline">インスタンスのバックアップ</v-card-title>
                <v-card-text>
                    {{ body }}
                    <v-form>
                        <v-checkbox
                                label="再起動しない"
                                v-model="noReboot"
                        >
                            <v-tooltip top slot="append">
                                <v-icon slot="activator">mdi-information</v-icon>
                                <span>
                                    有効な場合、Amazon EC2 はイメージの作成前にインスタンスをシャットダウンしません。<br>
                                    このオプションを使用すると、作成したイメージのファイルシステムの完全性は保証できません。
                                </span>
                            </v-tooltip>
                        </v-checkbox>
                    </v-form>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" flat
                           @click="cancel"
                           :disabled="isProgress">キャンセル
                    </v-btn>
                    <v-btn color="blue darken-1" flat
                           @click="confirm"
                           :loading="isProgress"
                           :disabled="isProgress">作成
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-layout>
</template>

<script>
  import {mapActions, mapGetters} from 'vuex'

  export default {
    data() {
      return {
        isOpen: false,
        isProgress: false,
        body: '',
        noReboot: false,
        resolve: null,
        reject: null
      }
    },
    computed: {
      ...mapGetters({
        resource: 'resourceDetail/resource'
      })
    },
    methods: {
      ...mapActions('resourceDetail', ['createResourceBackup']),
      open() {
        // モーダルを開く
        this.isProgress = false
        this.body = `${this.resource.name}のバックアップを作成します。`
        this.isOpen = true
        return new Promise((resolve, reject) => {
          this.resolve = resolve
          this.reject = reject
        })
      },
      confirm() {
        this.isProgress = true
        const data = {
          no_reboot: this.noReboot
        }
        this.createResourceBackup(data).then(() => {
          this.resolve(true)
        }).catch(() => {
          this.reject()
        }).finally(() => {
          this.isProgress = false
          this.isOpen = false
        })
      },
      cancel() {
        this.isProgress = false
        this.isOpen = false
        this.resolve(false)
      }
    }
  }
</script>