<template>
  <el-dialog :append-to-body="appendToBody" :visible.sync="dialogVisible" destroy-on-close @close="close">
    <div style="padding: 10px">
      <el-switch active-text="JSON-SCHEMA" v-model="item.jsonType" @change="formatChange" active-value="JSON-SCHEMA" />
    </div>
    <div v-if="codeEditActive">
      <ms-json-code-edit v-if="item.jsonType === 'JSON-SCHEMA'" :body="item" ref="jsonCodeEdit" />
      <ms-code-edit
        v-else
        :read-only="isReadOnly"
        :data.sync="item.value"
        :mode="'json'"
        height="400px"
        ref="codeEdit" />
    </div>
  </el-dialog>
</template>

<script>
import MsCodeEdit from 'metersphere-frontend/src/components/MsCodeEdit';
import Convert from '@/business/commons/json-schema/convert/convert';
import MsJsonCodeEdit from '@/business/commons/json-schema/JsonSchemaEditor';
import { parse } from 'lossless-json';

export default {
  name: 'MsApiVariableJson',
  components: { MsJsonCodeEdit, MsCodeEdit },
  props: {
    isReadOnly: {
      type: Boolean,
      default: false,
    },
    appendToBody: {
      type: Boolean,
      default() {
        return false;
      },
    },
  },
  data() {
    return {
      dialogVisible: false,
      jsonSchema: 'JSON',
      codeEditActive: true,
      item: {},
    };
  },
  watch: {
    'item.value'() {
      if (this.item.jsonType !== 'JSON-SCHEMA' && this.item.value) {
        try {
          const MsConvert = new Convert();
          let data = MsConvert.format(JSON.parse(this.item.value));
          if (this.item.jsonSchema) {
            this.item.jsonSchema = this.deepAssign(this.item.jsonSchema, data);
          }
        } catch (ex) {
          this.item.jsonSchema = '';
        }
      }
    },
  },

  methods: {
    open(item) {
      this.item.value = item.value;
      this.dialogVisible = true;
      this.reloadCodeEdit();
    },
    reloadCodeEdit() {
      this.codeEditActive = false;
      this.$nextTick(() => {
        this.codeEditActive = true;
      });
    },
    formatChange() {
      const MsConvert = new Convert();
      if (this.item.jsonType === 'JSON-SCHEMA') {
        if (this.item.value) {
          try {
              const jsonObj = parse(this.item.value)
              this.item.jsonSchema = MsConvert.format(jsonObj);
          } catch (e) {
            this.body.format = 'JSON';
            this.$message.error(this.$t('api_definition.body.json_format_error'));
          }
        }
      } else {
        if (this.item.jsonSchema) {
          MsConvert.schemaToJsonStr(this.item.jsonSchema, (result) => {
            if (result === 'Error') {
              this.body.format = 'JSON-SCHEMA';
              this.$message.error(this.$t('api_definition.body.json_format_error_tips'));
            } else {
              this.$set(this.item, 'value', result);
              this.$emit('callback', result);
            }
          });
        }
      }
    },
    saveAdvanced() {
      this.dialogVisible = false;
      if (this.item.jsonType === 'JSON-SCHEMA') {
        this.item.jsonType = 'JSON';
        this.formatChange();
      } else {
        this.$emit('callback', this.item.value);
      }
      this.item = {};
      this.reloadCodeEdit();
    },
    close() {
      this.saveAdvanced();
    },
  },
};
</script>
