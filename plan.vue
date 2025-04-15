<template>
  <div class="plans-container">
    <!-- 顶部统计卡片 -->
    <div class="stats-row">
      <el-card class="stat-card" :body-style="{ padding: '0px' }">
        <div class="stat-inner">
          <div class="stat-icon">
            <el-icon><Calendar /></el-icon>
          </div>
          <div class="stat-content">
            <div class="stat-value pulse">{{ activePlansCount }}</div>
            <div class="stat-label">进行中计划</div>
          </div>
        </div>
      </el-card>
      
      <el-card class="stat-card" :body-style="{ padding: '0px' }">
        <div class="stat-inner">
          <div class="stat-icon green">
            <el-icon><Check /></el-icon>
          </div>
          <div class="stat-content">
            <div class="stat-value pulse">{{ completedPlansCount }}</div>
            <div class="stat-label">已完成计划</div>
          </div>
        </div>
      </el-card>
      
      <el-card class="stat-card" :body-style="{ padding: '0px' }">
        <div class="stat-inner">
          <div class="stat-icon purple">
            <el-icon><Timer /></el-icon>
          </div>
          <div class="stat-content">
            <div class="stat-value pulse">{{ totalReadingMinutes }}</div>
            <div class="stat-label">总计划时长(分钟)</div>
          </div>
        </div>
      </el-card>
    </div>
    
    <!-- 进度条展示 -->
    <el-card class="progress-card">
      <template #header>
        <div class="card-header">
          <span>阅读计划完成进度</span>
        </div>
      </template>
      <div class="progress-container">
        <el-progress 
          :percentage="overallProgress" 
          :stroke-width="20"
          :show-text="false"
          class="animated-progress"
        ></el-progress>
        <div class="progress-text">{{ overallProgress }}%</div>
      </div>
    </el-card>

    <el-card class="plans-list-card">
      <template #header>
        <div class="card-header">
          <span>阅读计划</span>
          <el-button type="primary" @click="handleAdd" class="add-button">
            <el-icon><Plus /></el-icon> 新建计划
          </el-button>
        </div>
      </template>
      
      <div v-if="plans.length === 0" class="empty-plans">
        <el-empty description="暂无阅读计划">
          <div class="empty-hint">制定一个阅读计划，开始你的阅读之旅吧！</div>
          <el-button type="primary" @click="handleAdd" class="empty-button">
            <el-icon><Plus /></el-icon> 创建第一个计划
          </el-button>
        </el-empty>
      </div>
      
      <el-table 
        v-else
        :data="plans" 
        style="width: 100%"
        :row-class-name="tableRowClassName"
      >
        <el-table-column prop="title" label="计划名称">
          <template #default="scope">
            <div class="plan-title-cell">
              <el-icon v-if="isPlanActive(scope.row)" class="status-icon active"><Loading /></el-icon>
              <el-icon v-else-if="isPlanCompleted(scope.row)" class="status-icon completed"><Check /></el-icon>
              <el-icon v-else class="status-icon future"><Calendar /></el-icon>
              <span>{{ scope.row.title }}</span>
              <el-tag v-if="isPlanActive(scope.row)" type="success" size="small" effect="dark" class="plan-tag">进行中</el-tag>
              <el-tag v-else-if="isPlanCompleted(scope.row)" type="info" size="small" effect="dark" class="plan-tag">已完成</el-tag>
              <el-tag v-else type="warning" size="small" effect="dark" class="plan-tag">未开始</el-tag>
            </div>
          </template>
        </el-table-column>
        
        <el-table-column prop="targetMinutes" label="目标时长 (分钟)">
          <template #default="scope">
            <div class="highlight-text">{{ scope.row.targetMinutes }} 分钟</div>
          </template>
        </el-table-column>
        
        <el-table-column prop="startDate" label="开始日期" />
        <el-table-column prop="endDate" label="结束日期" />
        
        <el-table-column label="进度" width="200">
          <template #default="scope">
            <el-progress 
              :percentage="getPlanProgress(scope.row)" 
              :status="getPlanProgressStatus(scope.row)"
            ></el-progress>
          </template>
        </el-table-column>
        
        <el-table-column label="操作" width="200">
          <template #default="scope">
            <div class="action-buttons">
              <el-button size="small" type="primary" plain @click="handleEdit(scope.row)" class="action-button">
                <el-icon><Edit /></el-icon>
              </el-button>
              <el-button size="small" type="danger" plain @click="handleDelete(scope.row.id)" class="action-button">
                <el-icon><Delete /></el-icon>
              </el-button>
              <el-button size="small" type="success" plain @click="markAsCompleted(scope.row)" v-if="!isPlanCompleted(scope.row)" class="action-button">
                <el-icon><Check /></el-icon>
              </el-button>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加/编辑对话框 -->
    <el-dialog
      :title="dialogTitle"
      v-model="dialogVisible"
      width="50%"
      class="cyber-dialog"
      :close-on-click-modal="false"
    >
      <el-form :model="planForm" label-width="120px">
        <el-form-item label="计划名称">
          <el-input v-model="planForm.title" placeholder="例如：《三体》阅读计划"></el-input>
        </el-form-item>
        <el-form-item label="目标时长">
          <el-input-number v-model="planForm.targetMinutes" :min="1" placeholder="每天计划阅读的分钟数"></el-input-number>
        </el-form-item>
        <el-form-item label="开始日期">
          <el-date-picker
            v-model="planForm.startDate"
            type="date"
            placeholder="选择开始日期"
          />
        </el-form-item>
        <el-form-item label="结束日期">
          <el-date-picker
            v-model="planForm.endDate"
            type="date"
            placeholder="选择结束日期"
          />
        </el-form-item>
        <el-form-item label="书籍">
          <el-select v-model="planForm.bookId" placeholder="选择关联的书籍">
            <el-option
              v-for="book in books"
              :key="book.id"
              :label="book.title"
              :value="book.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="备注">
          <el-input
            type="textarea"
            v-model="planForm.notes"
            :rows="4"
            placeholder="添加一些备注信息，例如阅读目标、阅读范围等"
          ></el-input>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="handleSubmit">确认</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex'
import { ref, computed } from 'vue'
import { 
  Calendar, 
  Check, 
  Timer, 
  Delete, 
  Edit, 
  Plus, 
  Loading
} from '@element-plus/icons-vue'

export default {
  name: 'Plans',
  components: {
    Calendar, 
    Check, 
    Timer, 
    Delete, 
    Edit, 
    Plus, 
    Loading
  },
  data() {
    return {
      dialogVisible: false,
      dialogTitle: '新建计划',
      planForm: {
        title: '',
        targetMinutes: 30,
        startDate: '',
        endDate: '',
        bookId: '',
        notes: '',
        progress: 0
      },
      currentPlanId: null,
      // 模拟数据
      mockPlans: [
        { 
          id: 1, 
          title: '《三体》阅读计划', 
          targetMinutes: 60, 
          startDate: '2025-04-01', 
          endDate: '2025-04-30',
          bookId: 1,
          notes: '每天阅读一章',
          progress: 75
        },
        { 
          id: 2, 
          title: '《活着》阅读计划', 
          targetMinutes: 45, 
          startDate: '2025-05-01', 
          endDate: '2025-05-15',
          bookId: 2,
          notes: '细细品味余华的文笔',
          progress: 0
        },
        { 
          id: 3, 
          title: '《百年孤独》阅读计划', 
          targetMinutes: 30, 
          startDate: '2025-03-15', 
          endDate: '2025-03-30',
          bookId: 3,
          notes: '魔幻现实主义经典',
          progress: 100
        }
      ],
      mockBooks: [
        { id: 1, title: '三体' },
        { id: 2, title: '活着' },
        { id: 3, title: '百年孤独' },
        { id: 4, title: '围城' },
        { id: 5, title: '红楼梦' }
      ]
    }
  },
  computed: {
    ...mapState({
      storePlans: state => state.plans || [],
      storeBooks: state => state.books || []
    }),
    plans() {
      return this.storePlans.length > 0 ? this.storePlans : this.mockPlans;
    },
    books() {
      return this.storeBooks.length > 0 ? this.storeBooks : this.mockBooks;
    },
    activePlansCount() {
      return this.plans.filter(plan => this.isPlanActive(plan)).length;
    },
    completedPlansCount() {
      return this.plans.filter(plan => this.isPlanCompleted(plan)).length;
    },
    totalReadingMinutes() {
      return this.plans.reduce((sum, plan) => sum + (plan.targetMinutes || 0), 0);
    },
    overallProgress() {
      if (this.plans.length === 0) return 0;
      
      const totalProgress = this.plans.reduce((sum, plan) => sum + (plan.progress || 0), 0);
      return Math.round(totalProgress / this.plans.length);
    }
  },
  methods: {
    ...mapActions(['fetchPlan', 'updatePlan', 'deletePlan']),
    tableRowClassName({row, rowIndex}) {
      if (this.isPlanActive(row)) {
        return 'active-plan-row';
      } else if (this.isPlanCompleted(row)) {
        return 'completed-plan-row';
      }
      return '';
    },
    isPlanActive(plan) {
      const today = new Date();
      const startDate = new Date(plan.startDate);
      const endDate = new Date(plan.endDate);
      
      return startDate <= today && today <= endDate && (plan.progress || 0) < 100;
    },
    isPlanCompleted(plan) {
      return (plan.progress || 0) === 100;
    },
    getPlanProgress(plan) {
      return plan.progress || 0;
    },
    getPlanProgressStatus(plan) {
      if (this.isPlanCompleted(plan)) {
        return 'success';
      } else if (this.isPlanActive(plan)) {
        return '';
      }
      return 'warning';
    },
    handleAdd() {
      this.dialogTitle = '新建计划'
      this.planForm = {
        title: '',
        targetMinutes: 30,
        startDate: new Date(),
        endDate: this.getDefaultEndDate(),
        bookId: '',
        notes: '',
        progress: 0
      }
      this.currentPlanId = null
      this.dialogVisible = true
    },
    async handleEdit(plan) {
      this.dialogTitle = '编辑计划'
      this.planForm = { ...plan }
      this.currentPlanId = plan.id
      this.dialogVisible = true
    },
    async handleDelete(planId) {
      try {
        await this.$confirm('确定要删除这个阅读计划吗？', '警告', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        
        // 如果使用的是模拟数据，直接从本地删除
        if (this.storePlans.length === 0) {
          const index = this.mockPlans.findIndex(plan => plan.id === planId);
          if (index !== -1) {
            this.mockPlans.splice(index, 1);
          }
        } else {
          await this.deletePlan(planId);
        }
        
        this.$message.success('删除成功')
        this.animateStats();
      } catch (error) {
        if (error !== 'cancel') {
          this.$message.error('删除失败')
        }
      }
    },
    async markAsCompleted(plan) {
      try {
        const updatedPlan = { ...plan, progress: 100 };
        
        // 如果使用的是模拟数据，直接更新本地数据
        if (this.storePlans.length === 0) {
          const index = this.mockPlans.findIndex(p => p.id === plan.id);
          if (index !== -1) {
            this.mockPlans[index] = updatedPlan;
          }
        } else {
          await this.updatePlan({
            planId: plan.id,
            data: updatedPlan
          });
        }
        
        this.$message.success('计划已标记为完成！')
        this.animateStats();
      } catch (error) {
        this.$message.error('操作失败')
      }
    },
    async handleSubmit() {
      try {
        const formattedStartDate = this.formatDate(this.planForm.startDate);
        const formattedEndDate = this.formatDate(this.planForm.endDate);
        
        const updatedPlanForm = {
          ...this.planForm,
          startDate: formattedStartDate,
          endDate: formattedEndDate
        };
        
        if (this.currentPlanId) {
          // 如果使用的是模拟数据，直接更新本地数据
          if (this.storePlans.length === 0) {
            const index = this.mockPlans.findIndex(plan => plan.id === this.currentPlanId);
            if (index !== -1) {
              this.mockPlans[index] = { ...updatedPlanForm, id: this.currentPlanId };
            }
          } else {
            await this.updatePlan({
              planId: this.currentPlanId,
              data: updatedPlanForm
            });
          }
        } else {
          // 新增记录
          if (this.storePlans.length === 0) {
            // 为模拟数据生成新的 ID
            const newId = this.mockPlans.length > 0 
              ? Math.max(...this.mockPlans.map(plan => plan.id)) + 1 
              : 1;
            this.mockPlans.push({ ...updatedPlanForm, id: newId, progress: 0 });
          } else {
            // 通过 store 创建记录
            // 实现 createPlan action 调用
          }
        }
        
        this.dialogVisible = false;
        this.$message.success('保存成功');
        this.animateStats();
      } catch (error) {
        this.$message.error('保存失败');
      }
    },
    formatDate(date) {
      if (!date) return '';
      if (typeof date === 'string') return date;
      
      const d = new Date(date);
      const year = d.getFullYear();
      const month = String(d.getMonth() + 1).padStart(2, '0');
      const day = String(d.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    },
    getDefaultEndDate() {
      const date = new Date();
      date.setDate(date.getDate() + 30); // 默认30天后结束
      return date;
    },
    animateStats() {
      const elements = document.querySelectorAll('.pulse');
      elements.forEach(el => {
        el.classList.add('animate');
        setTimeout(() => {
          el.classList.remove('animate');
        }, 1000);
      });
    }
  },
  mounted() {
    // 尝试获取计划数据
    this.fetchPlan && this.fetchPlan();
    
    // 初始触发一次数字动画
    setTimeout(() => {
      this.animateStats();
    }, 500);
  }
}
</script>

<style scoped>
.plans-container {
  padding: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stats-row {
  display: flex;
  justify-content: space-between;
  gap: 20px;
  margin-bottom: 20px;
}

.stat-card {
  flex: 1;
  border-radius: 10px;
  overflow: hidden;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transform-origin: center;
}

.stat-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}

.stat-inner {
  display: flex;
  padding: 20px;
  background: linear-gradient(135deg, #1a1f3e, #162035);
  color: white;
}

.stat-icon {
  background: linear-gradient(135deg, #409EFF, #53a8ff);
  width: 60px;
  height: 60px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 15px;
  font-size: 24px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.stat-icon.green {
  background: linear-gradient(135deg, #67C23A, #85ce61);
}

.stat-icon.purple {
  background: linear-gradient(135deg, #8e44ad, #9b59b6);
}

.stat-content {
  flex: 1;
}

.stat-value {
  font-size: 28px;
  font-weight: bold;
  margin-bottom: 5px;
}

.stat-label {
  opacity: 0.8;
  font-size: 14px;
}

.progress-card {
  margin-bottom: 20px;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.progress-container {
  height: 60px;
  padding: 10px 20px;
  display: flex;
  align-items: center;
  position: relative;
}

.animated-progress {
  flex: 1;
}

.progress-text {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  font-size: 18px;
  font-weight: bold;
  color: white;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

.plans-list-card {
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.cyber-dialog {
  border-radius: 8px;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

.empty-plans {
  padding: 40px 0;
  text-align: center;
}

.empty-hint {
  color: #909399;
  margin: 15px 0;
}

.empty-button {
  margin-top: 15px;
}

.plan-title-cell {
  display: flex;
  align-items: center;
  gap: 8px;
}

.status-icon {
  font-size: 16px;
}

.status-icon.active {
  color: #409EFF;
  animation: spin 1.5s linear infinite;
}

.status-icon.completed {
  color: #67C23A;
}

.status-icon.future {
  color: #E6A23C;
}

.plan-tag {
  margin-left: 8px;
}

.highlight-text {
  font-weight: bold;
  color: #409EFF;
}

.action-buttons {
  display: flex;
  gap: 5px;
}

.action-button {
  transition: all 0.2s ease;
}

.action-button:hover {
  transform: scale(1.1);
}

.add-button {
  display: flex;
  align-items: center;
  gap: 5px;
  transition: all 0.3s ease;
}

.add-button:hover {
  transform: scale(1.05);
}

/* 表格行样式 */
:deep(.active-plan-row) {
  background-color: rgba(64, 158, 255, 0.1);
}

:deep(.completed-plan-row) {
  background-color: rgba(103, 194, 58, 0.1);
}

:deep(.el-table .active-plan-row:hover > td) {
  background-color: rgba(64, 158, 255, 0.2) !important;
}

:deep(.el-table .completed-plan-row:hover > td) {
  background-color: rgba(103, 194, 58, 0.2) !important;
}

/* 动画效果 */
.pulse.animate {
  animation: pulse 1s ease-out;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.2);
    color: #F56C6C;
  }
  100% {
    transform: scale(1);
  }
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .stats-row {
    flex-direction: column;
    gap: 10px;
  }
  
  .stat-card {
    width: 100%;
  }
}
</style>
