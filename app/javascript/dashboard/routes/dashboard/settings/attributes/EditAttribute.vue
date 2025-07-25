<script>
import { useVuelidate } from '@vuelidate/core';
import { useAlert } from 'dashboard/composables';
import { required, minLength } from '@vuelidate/validators';
import { getRegexp } from 'shared/helpers/Validators';
import { ATTRIBUTE_TYPES } from './constants';
import NextButton from 'dashboard/components-next/button/Button.vue';

export default {
  components: {
    NextButton,
  },
  props: {
    selectedAttribute: {
      type: Object,
      default: () => {},
    },
    isUpdating: {
      type: Boolean,
      default: false,
    },
  },
  emits: ['onClose'],
  setup() {
    return { v$: useVuelidate() };
  },
  data() {
    return {
      displayName: '',
      description: '',
      attributeType: 0,
      regexPattern: null,
      regexCue: null,
      regexEnabled: false,
      show: true,
      attributeKey: '',
      values: [],
      options: [],
      isTouched: true,
    };
  },
  validations: {
    displayName: {
      required,
    },
    attributeType: {
      required,
    },
    description: {
      required,
      minLength: minLength(1),
    },
    attributeKey: {
      required,
      isKey(value) {
        return !(value.indexOf(' ') >= 0);
      },
    },
  },
  computed: {
    types() {
      return ATTRIBUTE_TYPES.map(item => ({
        ...item,
        option: this.$t(`ATTRIBUTES_MGMT.ATTRIBUTE_TYPES.${item.key}`),
      }));
    },
    setAttributeListValue() {
      return this.selectedAttribute.attribute_values.map(values => ({
        name: values,
      }));
    },
    updatedAttributeListValues() {
      return this.values.map(item => item.name);
    },
    isButtonDisabled() {
      return this.v$.description.$invalid || this.isMultiselectInvalid;
    },
    isMultiselectInvalid() {
      return (
        this.isAttributeTypeList && this.isTouched && this.values.length === 0
      );
    },

    pageTitle() {
      return `${this.$t('ATTRIBUTES_MGMT.EDIT.TITLE')} - ${
        this.selectedAttribute.attribute_display_name
      }`;
    },
    selectedAttributeType() {
      return this.types.find(
        item =>
          item.key.toLowerCase() ===
          this.selectedAttribute.attribute_display_type
      )?.id;
    },
    keyErrorMessage() {
      if (!this.v$.attributeKey.isKey) {
        return this.$t('ATTRIBUTES_MGMT.ADD.FORM.KEY.IN_VALID');
      }
      return this.$t('ATTRIBUTES_MGMT.ADD.FORM.KEY.ERROR');
    },
    isAttributeTypeList() {
      return this.attributeType === 6;
    },
    isAttributeTypeText() {
      return this.attributeType === 0;
    },
    isRegexEnabled() {
      return this.regexEnabled;
    },
  },
  mounted() {
    this.setFormValues();
  },
  methods: {
    onClose() {
      this.$emit('onClose');
    },
    addTagValue(tagValue) {
      const tag = {
        name: tagValue,
      };
      this.values.push(tag);
      this.$refs.tagInput.$el.focus();
    },
    setFormValues() {
      const regexPattern = this.selectedAttribute.regex_pattern
        ? getRegexp(this.selectedAttribute.regex_pattern).source
        : null;
      this.displayName = this.selectedAttribute.attribute_display_name;
      this.description = this.selectedAttribute.attribute_description;
      this.attributeType = this.selectedAttributeType;
      this.attributeKey = this.selectedAttribute.attribute_key;
      this.regexPattern = regexPattern;
      this.regexCue = this.selectedAttribute.regex_cue;
      this.regexEnabled = regexPattern != null;
      this.values = this.setAttributeListValue;
    },
    async editAttributes() {
      this.v$.$touch();
      if (this.v$.$invalid) {
        return;
      }
      if (!this.regexEnabled) {
        this.regexPattern = null;
        this.regexCue = null;
      }
      try {
        await this.$store.dispatch('attributes/update', {
          id: this.selectedAttribute.id,
          attribute_description: this.description,
          attribute_display_name: this.displayName,
          attribute_values: this.updatedAttributeListValues,
          regex_pattern: this.regexPattern
            ? new RegExp(this.regexPattern).toString()
            : null,
          regex_cue: this.regexCue,
        });
        this.alertMessage = this.$t('ATTRIBUTES_MGMT.EDIT.API.SUCCESS_MESSAGE');
        this.onClose();
      } catch (error) {
        const errorMessage = error?.message;
        this.alertMessage =
          errorMessage || this.$t('ATTRIBUTES_MGMT.EDIT.API.ERROR_MESSAGE');
      } finally {
        useAlert(this.alertMessage);
      }
    },
    toggleRegexEnabled() {
      this.regexEnabled = !this.regexEnabled;
    },
  },
};
</script>

<template>
  <div class="flex flex-col h-auto overflow-auto">
    <woot-modal-header :header-title="pageTitle" />
    <form class="flex flex-col w-full" @submit.prevent="editAttributes">
      <div class="w-full">
        <woot-input
          v-model="displayName"
          :label="$t('ATTRIBUTES_MGMT.ADD.FORM.NAME.LABEL')"
          type="text"
          :class="{ error: v$.displayName.$error }"
          :error="
            v$.displayName.$error
              ? $t('ATTRIBUTES_MGMT.ADD.FORM.NAME.ERROR')
              : ''
          "
          :placeholder="$t('ATTRIBUTES_MGMT.ADD.FORM.NAME.PLACEHOLDER')"
          @blur="v$.displayName.$touch"
        />
        <woot-input
          v-model="attributeKey"
          :label="$t('ATTRIBUTES_MGMT.ADD.FORM.KEY.LABEL')"
          type="text"
          :class="{ error: v$.attributeKey.$error }"
          :error="v$.attributeKey.$error ? keyErrorMessage : ''"
          :placeholder="$t('ATTRIBUTES_MGMT.ADD.FORM.KEY.PLACEHOLDER')"
          readonly
          @blur="v$.attributeKey.$touch"
        />
        <label :class="{ error: v$.description.$error }">
          {{ $t('ATTRIBUTES_MGMT.ADD.FORM.DESC.LABEL') }}
          <textarea
            v-model="description"
            rows="5"
            type="text"
            :placeholder="$t('ATTRIBUTES_MGMT.ADD.FORM.DESC.PLACEHOLDER')"
            @blur="v$.description.$touch"
          />
          <span v-if="v$.description.$error" class="message">
            {{ $t('ATTRIBUTES_MGMT.ADD.FORM.DESC.ERROR') }}
          </span>
        </label>
        <label :class="{ error: v$.attributeType.$error }">
          {{ $t('ATTRIBUTES_MGMT.ADD.FORM.TYPE.LABEL') }}
          <select v-model="attributeType" disabled>
            <option v-for="type in types" :key="type.id" :value="type.id">
              {{ type.option }}
            </option>
          </select>
          <span v-if="v$.attributeType.$error" class="message">
            {{ $t('ATTRIBUTES_MGMT.ADD.FORM.TYPE.ERROR') }}
          </span>
        </label>
        <div v-if="isAttributeTypeList" class="multiselect--wrap">
          <label>
            {{ $t('ATTRIBUTES_MGMT.EDIT.TYPE.LIST.LABEL') }}
          </label>
          <multiselect
            ref="tagInput"
            v-model="values"
            :placeholder="$t('ATTRIBUTES_MGMT.ADD.FORM.TYPE.LIST.PLACEHOLDER')"
            label="name"
            track-by="name"
            :class="{ invalid: isMultiselectInvalid }"
            :options="options"
            multiple
            taggable
            @tag="addTagValue"
          />
          <label
            v-show="isMultiselectInvalid"
            class="text-n-ruby-9 dark:text-n-ruby-9 text-sm font-normal mt-1"
          >
            {{ $t('ATTRIBUTES_MGMT.ADD.FORM.TYPE.LIST.ERROR') }}
          </label>
        </div>
        <div v-if="isAttributeTypeText">
          <input
            v-model="regexEnabled"
            type="checkbox"
            @input="toggleRegexEnabled"
          />
          {{ $t('ATTRIBUTES_MGMT.ADD.FORM.ENABLE_REGEX.LABEL') }}
        </div>
        <woot-input
          v-if="isAttributeTypeText && isRegexEnabled"
          v-model="regexPattern"
          :label="$t('ATTRIBUTES_MGMT.ADD.FORM.REGEX_PATTERN.LABEL')"
          type="text"
          :placeholder="
            $t('ATTRIBUTES_MGMT.ADD.FORM.REGEX_PATTERN.PLACEHOLDER')
          "
        />
        <woot-input
          v-if="isAttributeTypeText && isRegexEnabled"
          v-model="regexCue"
          :label="$t('ATTRIBUTES_MGMT.ADD.FORM.REGEX_CUE.LABEL')"
          type="text"
          :placeholder="$t('ATTRIBUTES_MGMT.ADD.FORM.REGEX_CUE.PLACEHOLDER')"
        />
      </div>
      <div class="flex flex-row justify-end w-full gap-2 px-0 py-2">
        <NextButton
          faded
          slate
          type="reset"
          :label="$t('ATTRIBUTES_MGMT.ADD.CANCEL_BUTTON_TEXT')"
          @click.prevent="onClose"
        />
        <NextButton
          type="submit"
          :label="$t('ATTRIBUTES_MGMT.EDIT.UPDATE_BUTTON_TEXT')"
          :is-loading="isUpdating"
          :disabled="isButtonDisabled"
        />
      </div>
    </form>
  </div>
</template>

<style lang="scss" scoped>
.key-value {
  padding: 0 0.5rem 0.5rem 0;
  font-family: monospace;
}

.multiselect--wrap {
  margin-bottom: 1rem;
}

::v-deep {
  .multiselect {
    margin-bottom: 0;
  }

  .multiselect__content-wrapper {
    display: none;
  }

  .multiselect--active .multiselect__tags {
    border-radius: 0.3125rem;
  }
}
</style>
