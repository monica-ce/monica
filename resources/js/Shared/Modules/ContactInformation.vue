<template>
  <div class="mb-10">
    <!-- title + cta -->
    <div class="mb-3 items-center justify-between border-b border-gray-200 pb-2 dark:border-gray-700 sm:flex">
      <div class="mb-2 sm:mb-0 flex items-center gap-2">
        <Headset class="h-4 w-4 text-gray-600" />

        <span class="font-semibold">
          {{ $t('Contact information') }}
        </span>
      </div>
      <pretty-button
        :text="$t('Add a contact information')"
        :icon="'plus'"
        :class="'w-full sm:w-fit'"
        @click="showCreateContactInformationModal" />
    </div>

    <!-- add a contact information modal -->
    <form
      v-if="addContactInformationModalShown"
      class="mb-6 rounded-lg border border-gray-200 bg-gray-50 dark:border-gray-700 dark:bg-gray-900"
      @submit.prevent="submit()">
      <div class="border-b border-gray-200 dark:border-gray-700">
        <div v-if="form.errors.length > 0" class="p-5">
          <errors :errors="form.errors" />
        </div>

        <!-- name -->
        <div class="border-b border-gray-200 p-5 dark:border-gray-700">
          <text-input
            ref="newData"
            v-model="form.data"
            :label="$t('Content')"
            :type="'text'"
            :autofocus="true"
            :input-class="'block w-full'"
            :required="true"
            :autocomplete="false"
            :maxlength="255"
            @esc-key-pressed="addContactInformationModalShown = false" />
        </div>

        <div class="p-5">
          <!-- contact information types -->
          <dropdown
            v-model.number="form.contact_information_type_id"
            :data="data.contact_information_types"
            :required="true"
            :placeholder="$t('Choose a value')"
            :dropdown-class="'block w-full'"
            :label="$t('Type')" />
        </div>
      </div>

      <div class="flex justify-between p-5">
        <pretty-span :text="$t('Cancel')" :class="'me-3'" @click="addContactInformationModalShown = false" />
        <pretty-button :text="$t('Save')" :state="loadingState" :icon="'plus'" :class="'save'" />
      </div>
    </form>

    <!-- contact infos -->
    <div v-if="localContactInformation.length > 0">
      <ul class="mb-4 rounded-lg border border-gray-200 bg-white dark:border-gray-700 dark:bg-gray-900">
        <li
          v-for="info in localContactInformation"
          :key="info.id"
          class="item-list border-b border-gray-200 hover:bg-slate-50 dark:border-gray-700 dark:bg-slate-900 dark:hover:bg-slate-800">
          <!-- contact information -->
          <div v-if="editedContactInformationId !== info.id" class="flex items-center justify-between px-3 py-2">
            <div>
              <a :href="info.data_with_protocol" class="text-blue-500 hover:underline">{{ info.data }}</a>
              <span class="ms-2 text-xs text-gray-500">({{ info.label }})</span>
            </div>

            <!-- actions -->
            <ul class="text-sm">
              <li
                class="me-4 inline cursor-pointer text-blue-500 hover:underline"
                @click="showEditContactInformationModal(info)">
                {{ $t('Edit') }}
              </li>
              <li class="inline cursor-pointer text-red-500 hover:text-red-900" @click="destroy(info)">
                {{ $t('Delete') }}
              </li>
            </ul>
          </div>

          <!-- edit info modal -->
          <form
            v-if="editedContactInformationId === info.id"
            class="bg-gray-50 dark:bg-gray-900"
            @submit.prevent="update(info)">
            <div class="border-b border-gray-200 dark:border-gray-700">
              <div v-if="form.errors.length > 0" class="p-5">
                <errors :errors="form.errors" />
              </div>

              <!-- name -->
              <div class="border-b border-gray-200 p-5 dark:border-gray-700">
                <text-input
                  ref="rename"
                  v-model="form.data"
                  :label="$t('Content')"
                  :type="'text'"
                  :autofocus="true"
                  :input-class="'block w-full'"
                  :required="false"
                  :autocomplete="false"
                  :maxlength="255"
                  @esc-key-pressed="editedContactInformationId = 0" />
              </div>

              <div class="p-5">
                <!-- contact information types -->
                <dropdown
                  v-model.number="form.contact_information_type_id"
                  :data="data.contact_information_types"
                  :required="true"
                  :placeholder="$t('Choose a value')"
                  :dropdown-class="'block w-full'"
                  :label="$t('Type')" />
              </div>
            </div>

            <div class="flex justify-between p-5">
              <pretty-span :text="$t('Cancel')" :class="'me-3'" @click="editedContactInformationId = 0" />
              <pretty-button :text="$t('Save')" :state="loadingState" :icon="'check'" :class="'save'" />
            </div>
          </form>
        </li>
      </ul>
    </div>

    <!-- blank state -->
    <div
      v-if="localContactInformation.length === 0"
      class="mb-6 rounded-lg border border-gray-200 bg-white dark:border-gray-700 dark:bg-gray-900">
      <img src="/img/contact_blank_contact.svg" :alt="$t('Contact informations')" class="mx-auto mt-4 h-20 w-20" />
      <p class="px-5 pb-5 pt-2 text-center">
        {{ $t('There are no contact information yet.') }}
      </p>
    </div>
  </div>
</template>

<script>
import PrettyButton from '@/Shared/Form/PrettyButton.vue';
import PrettySpan from '@/Shared/Form/PrettySpan.vue';
import TextInput from '@/Shared/Form/TextInput.vue';
import Dropdown from '@/Shared/Form/Dropdown.vue';
import Errors from '@/Shared/Form/Errors.vue';
import { Headset } from 'lucide-vue-next';

export default {
  components: {
    PrettyButton,
    PrettySpan,
    TextInput,
    Dropdown,
    Errors,
    Headset,
  },

  props: {
    data: {
      type: Object,
      default: null,
    },
  },

  data() {
    return {
      loadingState: '',
      addContactInformationModalShown: false,
      localContactInformation: [],
      editedContactInformationId: 0,
      form: {
        data: '',
        contact_information_type_id: 0,
        errors: [],
      },
    };
  },

  created() {
    this.localContactInformation = this.data.contact_information;
  },

  methods: {
    showCreateContactInformationModal() {
      this.addContactInformationModalShown = true;
      this.form.errors = [];
      this.form.data = '';
      this.form.contact_information_type_id = this.data.contact_information_types[0].id;

      this.$nextTick().then(() => {
        this.$refs.newData.focus();
      });
    },

    showEditContactInformationModal(info) {
      this.form.errors = [];
      this.editedContactInformationId = info.id;
      this.form.contact_information_type_id = info.contact_information_type.id;
      this.form.data = info.data;

      this.$nextTick().then(() => {
        this.$refs.rename[0].focus();
      });
    },

    submit() {
      this.loadingState = 'loading';

      axios
        .post(this.data.url.store, this.form)
        .then((response) => {
          this.flash(this.$t('The contact information has been created'), 'success');
          this.localContactInformation.unshift(response.data.data);
          this.loadingState = '';
          this.addContactInformationModalShown = false;
        })
        .catch((error) => {
          this.loadingState = '';
          this.form.errors = error.response.data;
        });
    },

    update(info) {
      this.loadingState = 'loading';

      axios
        .put(info.url.update, this.form)
        .then((response) => {
          this.loadingState = '';
          this.flash(this.$t('The contact information has been edited'), 'success');
          this.localContactInformation[this.localContactInformation.findIndex((x) => x.id === info.id)] =
            response.data.data;
          this.editedContactInformationId = 0;
        })
        .catch((error) => {
          this.loadingState = '';
          this.form.errors = error.response.data;
        });
    },

    destroy(info) {
      if (confirm(this.$t('Are you sure? This will delete the contact information permanently.'))) {
        axios
          .delete(info.url.destroy)
          .then(() => {
            this.flash(this.$t('The contact information has been deleted'), 'success');
            var id = this.localContactInformation.findIndex((x) => x.id === info.id);
            this.localContactInformation.splice(id, 1);
          })
          .catch((error) => {
            this.loadingState = null;
            this.form.errors = error.response.data;
          });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.icon-sidebar {
  color: #737e8d;
  top: -2px;
}

.item-list {
  &:hover:first-child {
    border-top-left-radius: 8px;
    border-top-right-radius: 8px;
  }

  &:last-child {
    border-bottom: 0;
  }

  &:hover:last-child {
    border-bottom-left-radius: 8px;
    border-bottom-right-radius: 8px;
  }
}
</style>
