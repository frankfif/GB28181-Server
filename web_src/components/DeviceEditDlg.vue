<template>
    <FormDlg title="编辑设备" @hide="onHide" @show="onShow" @submit="onSubmit" ref="dlg" :disabled="errors.any()">
        <div :class="{'form-group':true,'has-error': errors.has('serial')}">
            <label for="input-serial" class="col-sm-4 control-label">设备编号
                <span class="text-red">*</span>
            </label>
            <div class="col-sm-7">
                <input type="text" class="form-control" readonly="readonly" id="input-serial" name="serial" v-model.trim="form.serial" data-vv-as="设备编号" v-validate="'required'">
            </div>
        </div>
        <div :class="{'form-group':true,'has-error': errors.has('name')}">
            <label for="input-name" class="col-sm-4 control-label">设备名称
            </label>
            <div class="col-sm-7">
                <input type="text" class="form-control" id="input-name" name="name" v-model.trim="form.name" data-vv-as="设备名称" @keydown.enter="$el.querySelector('#input-password').focus()">
            </div>
        </div>
    </FormDlg>
</template>

<script>
import FormDlg from 'components/FormDlg.vue'
import $ from 'jquery'

export default {
    data() {
        return {
            form: this.defForm(),
            smsList: [],
        }
    },
    components: {
        FormDlg
    },
    methods: {
        defForm() {
            return {
                serial: '',
                name: ''
            }
        },
        onHide() {
            this.form = this.defForm();
            this.$emit("hide");
        },
        onShow() {
            this.errors.clear();
            this.$emit("show");
        },
        fetchSMSList() {
            return new Promise((resolve, reject) => {
                $.get("/api/v1/sms/list").then(ret => {
                    resolve(ret.map(sms => ({
                        id: sms.Serial||"",
                        name: sms.Serial||"",
                    })));
                }).fail(() => {
                    resolve([]);
                })
            })
        },
        async onSubmit() {
            var ok = await this.$validator.validateAll();
            if(!ok) {
                var e = this.errors.items[0];
                this.$message({
                    type: 'error',
                    message: e.msg
                })
                $(`[name=${e.field}]`).focus();
                return;
            }
            $.get('/api/v1/device/setinfo', this.form).then(data => {
                this.$refs['dlg'].hide();
                this.$emit("submit");
            })
        },
        async show(data) {
            this.errors.clear();
            if(data) {
                Object.assign(this.form, data);
            }
            var smsList = await this.fetchSMSList();
            var smsid = this.form['sms_id'];
            if(smsid && !smsList.some(sms => (sms.id == smsid))) {
                smsList.push({
                    id: smsid,
                    name: `${smsid}(not found)`,
                });
            }
            this.smsList = smsList;
            if(!this.form['subscribe_interval']) {
                this.form['subscribe_interval'] = '';
            }
            this.$nextTick(() => {
                this.$refs['dlg'].show();
            })
        }
    }
}
</script>
