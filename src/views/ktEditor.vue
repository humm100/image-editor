<template>
  <div class="ui-index">
    <div class="ui-index_header">
      <div class="soft">方案设计</div>
      <div class="project actions">
        <el-button class="btn-action" type="text">保存</el-button>
        <el-dropdown>
          <span class="el-dropdown-link">
            清空<i class="el-icon-arrow-down el-icon--right"></i>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item>清空所有</el-dropdown-item>
            <el-dropdown-item>清空标注</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        <el-dropdown>
          <span class="el-dropdown-link">
            标注<i class="el-icon-arrow-down el-icon--right"></i>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item>添加标注</el-dropdown-item>
            <el-dropdown-item>一键标注</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        <el-dropdown>
          <span class="el-dropdown-link">
            标注颜色<i class="el-icon-arrow-down el-icon--right"></i>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item>red</el-dropdown-item>
            <el-dropdown-item>blue</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        <el-divider direction="vertical"></el-divider>
        <el-button class="btn-action" type="text">空间</el-button>
        <el-button class="btn-action" type="text">方案信息</el-button>
        <el-button class="btn-action" type="text">图纸清单</el-button>
        <el-button class="btn-action" type="text">生成方案</el-button>
        <el-divider direction="vertical"></el-divider>
        <el-button class="btn-action" type="text" @click="openFormwork">保存模板</el-button>
        <el-button class="btn-action" type="text" @click="importJson">导入JSON</el-button>
        <el-button class="btn-action" type="text" @click="downloadJson">下载JSON</el-button>
        <el-button class="btn-action" type="text" @click="downloadImg">下载图片</el-button>
      </div>
    </div>
    <div class="ui-index_content">
      <div class="ui-editor">
        <div class="ui-editor_formwork">
          <el-tabs type="border-card" v-model="activeName" >
            <el-tab-pane label="模板" name="template">
              <ul class="ui-editor_formwork_template">
                <li v-for="(item, index) in tempList" :key="index"
                  title="点击引用模板"
                 @click="loadFormwork(item)">
                  <img :src="item.img" alt="">
                  <div>{{item.title}}</div>
                  <div class="ui-editor_delete" @click="deleteFormwork($event, index, item)">
                    <i class="el-icon-delete"></i>
                  </div>
                </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="组件" name="material">
              <ul class="ui-editor_formwork_material">
                <li v-for="(item, index) in material" :key="index"
                dragtype="image"
                draggable
                @dragstart="handleDragStart($event,item)"
                title="拖拽添加"
                >
                  <template v-if="item.type == 'Image'">
                    <img :src="`${publicPath}${item.icon}`" alt="">
                  </template>
                  <template v-else>
                    <svg class="icon">
                      <use :xlink:href="item.icon"></use>
                    </svg>
                  </template>
                  <div>{{item.name}}</div>
                </li>
              </ul>
            </el-tab-pane>
          </el-tabs>
        </div>
        <div class="ui-editor_workspace">
          <div class="ui-editor_container"
            @drop="handleDrop"
            @dragover="handleDragOver"
            @contextmenu="showMenu"
            ref="ui-editor_container"
          >
            <canvas id="editor_canvas"></canvas>
          </div>
          <div class="ui-editor_menu" ref="editorMenu">
            <ul>
              <li @click="handleMenu(item)" v-for="(item, index) in menuList" :key="index">
                {{item.name}}
              </li>
            </ul>
          </div>
        </div>
        <div class="ui-editor_property">
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span>卡片名称</span>
            </div>
            <div>000</div>
          </el-card>
          <el-tabs type="border-card">
            <el-tab-pane label="设置">
              <div class="">
                <el-form
                class="ui-margin-top"
                label-width="50px"
              >
                <el-form-item label="宽度">
                  <el-input v-model="setting.width" @input="handleBackgroundSet"></el-input>
                </el-form-item>
                <el-form-item label="高度">
                  <el-input v-model="setting.height" @input="handleBackgroundSet"></el-input>
                </el-form-item>
                <el-form-item label="背景">
                  <div class="ui-editor_gradient">
                    <div class="ui-editor_gradient_item">
                    <el-color-picker
                      show-alpha
                      color-format="hsl/hsv/hex/rgb"
                      v-model="setting.background"
                      @change="handleBackgroundSet"
                    ></el-color-picker>
                    </div>
                    <div class="ui-editor_gradient_item"
                    v-for="(item, index) in backgroundGradient" :key="index"
                    @click="handleBackgroundGradient(item)"
                    :style="{
                      backgroundImage: item.background
                    }"
                    >
                    </div>
                  </div>
                </el-form-item>
              </el-form>
              </div>
            </el-tab-pane>
            <el-tab-pane label="属性">
              <el-form
                class="ui-margin-top"
                label-width="80px"
              >
                <el-form-item label="填充颜色">
                  <el-color-picker
                      size="mini"
                      show-alpha
                      color-format="hsl/hsv/hex/rgb"
                      v-model="editObject.fill"
                      @change="handleChangeObject({ fill: editObject.fill })"
                    ></el-color-picker>
                </el-form-item>
                <el-form-item label="描边颜色">
                  <el-color-picker
                      size="mini"
                      show-alpha
                      color-format="hsl/hsv/hex/rgb"
                      v-model="editObject.stroke"
                      @change="handleChangeObject({ stroke: editObject.stroke })"
                    ></el-color-picker>
                </el-form-item>
                <el-form-item label="描边大小">
                  <el-input
                      size="mini"
                      v-model="editObject.strokeWidth"
                      type="number"
                      @input="handleChangeObject({ strokeWidth: +editObject.strokeWidth })"
                    ></el-input>
                </el-form-item>
              </el-form>
            </el-tab-pane>
          </el-tabs>
        </div>
      </div>
    </div>
    <el-dialog
      title="保存模板"
      :visible.sync="tempVisible"
      width="30%">
      <el-form label-width="80px">
        <el-form-item label="模板名称">
          <el-input v-model="tempTitle"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button size="small" @click="closeTempDialog">取 消</el-button>
        <el-button size="small" type="primary" @click="saveFormwork">保 存</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import Index from './index'
export default Index
</script>
