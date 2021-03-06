<template>
  <div v-if="item.children && (!item.meta || !item.meta.hidden)" class="menu-wrapper">
    <template v-if="hasOneShowingChild(item.children) && !onlyOneChild.children && (!item.meta || !item.meta.alwaysShow)">
      <app-link :to="resolvePath(onlyOneChild.path)">
        <el-menu-item :index="resolvePath(onlyOneChild.path)" :class="{'submenu-title-noDropdown': !isNest}">
          <item v-if="onlyOneChild.meta" :icon="onlyOneChild.meta.icon || item.meta.icon" :title="onlyOneChild.meta.title" />
        </el-menu-item>
      </app-link>
    </template>
    <el-submenu v-else :index="resolvePath(item.path)">
      <template slot="title">
        <item v-if="item.meta" :icon="item.meta.icon" :title="item.meta.title" />
      </template>
      <template v-for="child in childrenFilter(item.children)">
        <sidebar-item
          v-if="child.children && child.children.length > 0"
          :is-nest="true"
          :item="child"
          :key="child.path"
          :base-path="resolvePath(child.path)"
          class="nest-menu"/>
        <app-link v-else :to="resolvePath(child.path)" :key="child.name">
          <el-menu-item :index="resolvePath(child.path)">
            <item v-if="child.meta" :icon="child.meta.icon" :title="child.meta.title" />
          </el-menu-item>
        </app-link>
      </template>
    </el-submenu>
  </div>
</template>

<script lang="ts">
import path from 'path';
import { Route } from 'vue-router';
import { isExternal } from '@/utils/validate';
import { Component, Vue, Prop } from 'vue-property-decorator';
import Item from './Item.vue';
import AppLink from './Link.vue';

@Component({
  // Set 'name' here to prevent uglifyjs from causing recursive component not work
  // See https://medium.com/haiiro-io/element-component-name-with-vue-class-component-f3b435656561 for detail
  name: 'SidebarItem',
  components: {
    Item,
    AppLink,
  },
})
export default class SidebarItem extends Vue {
  @Prop({ required: true }) private item!: Route;
  @Prop({ default: false }) private isNest!: boolean;
  @Prop({ default: '' }) private basePath!: string;

  private onlyOneChild: Route | null = null;

  private hasOneShowingChild(children: Route[]) {
    if (!children) { return false; }
    const showingChildren = children.filter((item: Route) => {
      if (item.meta && item.meta.hidden) {
        return false;
      } else {
        this.onlyOneChild = item; // This will only be used if hasOneShowingChild return true
        return true;
      }
    });
    return showingChildren.length === 1;
  }

  private resolvePath(routePath: string) {
    if (this.isExternalLink(routePath)) {
      return routePath;
    }
    return path.resolve(this.basePath, routePath);
  }

  private isExternalLink(routePath: string) {
    return isExternal(routePath);
  }

  private childrenFilter(children: Route[]) {
    return children.filter((child) => !child.meta || !child.meta.hidden);
  }
}
</script>
