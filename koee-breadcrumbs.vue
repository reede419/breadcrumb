<template>
  <div>
    <nav v-if="breadCrumbs.length">
      <ul class="koee-breadcrumbs koee-breadcrumbs-desktop pl-0">
        <li
          v-for="(crumb, i) in breadCrumbs"
          :key="i"
          class="koee-breadcrumb-item"
          :class="{ 'active-koee-breadcrumb': crumb.active }"
        >
          <b-button
            class="koee-breadcrumb-button"
            :disabled="crumb.active"
            variant="transparent"
            @click="clickOnBreadcrumb(crumb)"
          >
            {{ $te(`breadcrumbs.${crumb.key}`) ? $t(`breadcrumbs.${crumb.key}`) : crumb.key }}
          </b-button>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import boxNameQuery from '@/graphql/boxinfo/boxName.gql';

export default {
  name: 'koee-breadcrumbs',
  props: {
    additionalCrumbs: {
      type: Array,
    },
  },
  apollo: {
    boxName: {
      query: boxNameQuery,
      variables() {
        return {
          deviceId: this.currentDeviceId,
        };
      },
      update(data) {
        const queryData = data[Object.keys(data)[0]];
        return queryData?.boxes[0]?.digitalThing?.name || '';
      },
      skip() {
        return !this.currentDeviceId;
      },
    },
  },
  data() {
    return {};
  },
  computed: {
    currentDeviceId() {
      return this.$route.params.deviceId;
    },
    pathArray() {
      let routePathArray = this.$route.path.split('/');
      if (this.additionalCrumbs && this.additionalCrumbs.length) {
        routePathArray.push(...this.additionalCrumbs);
      }
      routePathArray.shift();
      return routePathArray;
    },
    breadCrumbs() {
      const breadCrumbs = [];
      let breadcrumb = '';
      for (let i = 0; i < this.pathArray.length; ++i) {
        breadcrumb = `${breadcrumb}${'/'}${this.pathArray[i]}`;
        breadCrumbs.push({
          href: breadcrumb,
          active: i + 1 === this.pathArray.length,
          key: this.pathArray[i] === this.currentDeviceId ? this.boxName : this.pathArray[i],
        });
      }
      return breadCrumbs;
    },
  },
  methods: {
    clickOnBreadcrumb(crumb) {
      if (this.$listeners && this.$listeners.clickOnBreadcrumb) {
        this.$emit('clickOnBreadcrumb', crumb, () => {
          this.clickOnBreadcrumbBase(crumb);
        });
        return;
      }
      this.clickOnBreadcrumbBase(crumb);
    },
    clickOnBreadcrumbBase(crumb) {
      this.$router.push({ path: crumb.href });
    },
  },
};
</script>

<style lang="scss" scoped>
.koee-breadcrumbs :deep {
  display: -ms-flexbox;
  display: flex;
  -ms-flex-wrap: wrap;
  flex-wrap: wrap;
  list-style: none;
  .koee-breadcrumb-item {
    .btn {
      padding: 0 !important;
      min-height: auto !important;
      color: var(--gray) !important;
      font-family: 'Manrope Medium', sans-serif !important;
      font-size: 1.25rem !important;
      text-transform: none !important;
      &.disabled {
        border: none !important;
        background-color: transparent !important;
        color: var(--white) !important;
        opacity: 1 !important;
      }
    }
  }
  .koee-breadcrumb-item + .koee-breadcrumb-item {
    padding-left: 0.5rem;
  }
  .koee-breadcrumb-item + .koee-breadcrumb-item:before {
    float: left;
    padding-right: 0.5rem;
    padding-top: 3px;
    color: var(--gray);
    font-size: larger;
    content: '/';
  }
  @media (max-width: 992px) {
    .koee-breadcrumb-item {
      display: none;
      &:nth-last-child(2) {
        display: block;
        position: relative;
        padding-left: 2rem;
        &:before {
          content: '';
        }
        .koee-breadcrumb-button {
          &:before {
            content: '';
            position: absolute;
            height: 20px;
            width: 20px;
            display: block;
            left: 0;
            top: 4px;
            background-image: url("data:image/svg+xml,%3Csvg width='20' height='20' viewBox='0 0 20 20' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M15.8327 10H4.16602' stroke='%23F4F4F4' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round'/%3E%3Cpath d='M9.99935 15.8333L4.16602 9.99996L9.99935 4.16663' stroke='%23F4F4F4' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/svg%3E%0A");
          }
        }
      }
    }
  }
}
</style>
