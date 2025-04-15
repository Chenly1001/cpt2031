<template>
  <div class="books-container">
    <!-- 统计卡片 -->
    <div class="stats-row">
      <el-card class="stat-card">
        <div class="stat-inner">
          <div class="stat-icon">
            <el-icon><Collection /></el-icon>
          </div>
          <div class="stat-content">
            <div class="stat-value pulse">{{ totalBooks }}</div>
            <div class="stat-label">书籍总数</div>
          </div>
        </div>
      </el-card>
      
      <el-card class="stat-card">
        <div class="stat-inner">
          <div class="stat-icon blue">
            <el-icon><User /></el-icon>
          </div>
          <div class="stat-content">
            <div class="stat-value pulse">{{ uniqueAuthors }}</div>
            <div class="stat-label">作者数量</div>
          </div>
        </div>
      </el-card>
      
      <el-card class="stat-card">
        <div class="stat-inner">
          <div class="stat-icon purple">
            <el-icon><Reading /></el-icon>
          </div>
          <div class="stat-content">
            <div class="stat-value pulse">{{ recentlyAdded }}</div>
            <div class="stat-label">最近新增</div>
          </div>
        </div>
      </el-card>
    </div>

    <el-card class="main-card">
      <template #header>
        <div class="card-header">
          <span class="title">书籍管理</span>
          <el-button type="primary" @click="handleAdd" class="add-button">
            <el-icon><Plus /></el-icon> 新增书籍
          </el-button>
        </div>
      </template>
      
      <!-- 搜索与筛选区域 -->
      <div class="search-filter-container">
        <div class="search-box">
          <el-input
            v-model="searchQuery"
            placeholder="搜索书名、作者、出版社..."
            clearable
            @clear="handleSearch"
            @input="handleSearch"
          >
            <template #prefix>
              <el-icon><Search /></el-icon>
            </template>
            <template #append>
              <el-button @click="handleSearch">搜索</el-button>
            </template>
          </el-input>
        </div>
        
        <div class="filter-section">
          <el-select v-model="authorFilter" clearable placeholder="按作者筛选" @change="handleFilter">
            <el-option
              v-for="author in authorOptions"
              :key="author"
              :label="author"
              :value="author"
            />
          </el-select>
          
          <el-select v-model="publisherFilter" clearable placeholder="按出版社筛选" @change="handleFilter">
            <el-option
              v-for="publisher in publisherOptions"
              :key="publisher"
              :label="publisher"
              :value="publisher"
            />
          </el-select>
          
          <el-select v-model="sortOption" placeholder="排序方式" @change="handleSort">
            <el-option label="按书名排序 (A-Z)" value="titleAsc" />
            <el-option label="按书名排序 (Z-A)" value="titleDesc" />
            <el-option label="按添加日期 (新-旧)" value="dateDesc" />
            <el-option label="按添加日期 (旧-新)" value="dateAsc" />
          </el-select>
          
          <el-button type="info" plain @click="resetFilters">
            <el-icon><Refresh /></el-icon> 重置
          </el-button>
        </div>
      </div>
      
      <!-- 书籍展示区域 -->
      <div v-if="filteredBooks.length === 0" class="empty-books">
        <el-empty description="暂无书籍">
          <div class="empty-hint">添加第一本书籍，开始你的阅读管理！</div>
          <el-button type="primary" @click="handleAdd" class="empty-button">
            <el-icon><Plus /></el-icon> 添加书籍
          </el-button>
        </el-empty>
      </div>
      
      <div v-else>
        <!-- 切换视图按钮 -->
        <div class="view-toggle">
          <el-radio-group v-model="viewMode" size="small">
            <el-radio-button label="table">
              <el-icon><List /></el-icon> 表格视图
            </el-radio-button>
            <el-radio-button label="card">
              <el-icon><Grid /></el-icon> 卡片视图
            </el-radio-button>
          </el-radio-group>
        </div>
        
        <!-- 表格视图 -->
        <el-table 
          v-if="viewMode === 'table'"
          :data="filteredBooks" 
          style="width: 100%" 
          stripe
          :row-class-name="tableRowClassName"
        >
          <el-table-column type="expand">
            <template #default="props">
              <div class="book-expand">
                <div class="book-cover" v-if="props.row.coverUrl">
                  <img :src="props.row.coverUrl" alt="封面" />
                </div>
                <div class="book-details">
                  <h3>{{ props.row.title }}</h3>
                  <p class="book-description">{{ props.row.description || '暂无简介' }}</p>
                  <div class="book-meta">
                    <span><el-icon><Calendar /></el-icon> 添加时间：{{ formatDate(props.row.createdAt) }}</span>
                    <span v-if="props.row.lastRead"><el-icon><Timer /></el-icon> 最近阅读：{{ formatDate(props.row.lastRead) }}</span>
                  </div>
                </div>
              </div>
            </template>
          </el-table-column>
          <el-table-column prop="title" label="书名" min-width="200">
            <template #default="scope">
              <div class="book-title-cell">
                <span class="book-title">{{ scope.row.title }}</span>
              </div>
            </template>
          </el-table-column>
          <el-table-column prop="author" label="作者" min-width="150" />
          <el-table-column prop="publisher" label="出版社" min-width="180" />
          <el-table-column prop="isbn" label="ISBN" min-width="150" />
          <el-table-column label="操作" width="200" fixed="right">
            <template #default="scope">
              <div class="action-buttons">
                <el-tooltip content="编辑" placement="top" :hide-after="300">
                  <el-button size="small" type="primary" circle @click="handleEdit(scope.row)">
                    <el-icon><Edit /></el-icon>
                  </el-button>
                </el-tooltip>
                <el-tooltip content="删除" placement="top" :hide-after="300">
                  <el-button size="small" type="danger" circle @click="handleDelete(scope.row.id)">
                    <el-icon><Delete /></el-icon>
                  </el-button>
                </el-tooltip>
                <el-tooltip content="创建阅读计划" placement="top" :hide-after="300">
                  <el-button size="small" type="success" circle @click="createReadingPlan(scope.row)">
                    <el-icon><DocumentAdd /></el-icon>
                  </el-button>
                </el-tooltip>
              </div>
            </template>
          </el-table-column>
        </el-table>
        
        <!-- 卡片视图 -->
        <div v-else class="book-cards">
          <el-card 
            v-for="book in filteredBooks" 
            :key="book.id" 
            class="book-card"
            :body-style="{ padding: '0px' }"
          >
            <div class="book-card-cover" :style="{ backgroundColor: getRandomColor(book.id) }">
              <span class="book-card-title">{{ book.title.charAt(0) }}</span>
            </div>
            <div class="book-card-content">
              <h3 class="book-card-name">{{ book.title }}</h3>
              <p class="book-card-author">{{ book.author }}</p>
              <p class="book-card-publisher">{{ book.publisher }}</p>
              <div class="book-card-actions">
                <el-tooltip content="编辑" placement="top" :hide-after="300">
                  <el-button size="small" type="primary" circle @click="handleEdit(book)">
                    <el-icon><Edit /></el-icon>
                  </el-button>
                </el-tooltip>
                <el-tooltip content="删除" placement="top" :hide-after="300">
                  <el-button size="small" type="danger" circle @click="handleDelete(book.id)">
                    <el-icon><Delete /></el-icon>
                  </el-button>
                </el-tooltip>
                <el-tooltip content="创建阅读计划" placement="top" :hide-after="300">
                  <el-button size="small" type="success" circle @click="createReadingPlan(book)">
                    <el-icon><DocumentAdd /></el-icon>
                  </el-button>
                </el-tooltip>
              </div>
            </div>
          </el-card>
        </div>
        
        <!-- 分页 -->
        <div class="pagination-container">
          <el-pagination
            v-model:current-page="currentPage"
            v-model:page-size="pageSize"
            :page-sizes="[10, 20, 50, 100]"
            layout="total, sizes, prev, pager, next, jumper"
            :total="filteredBooks.length"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
          />
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
import { ref, computed, nextTick, onMounted } from 'vue'
import { mapState, mapActions } from 'vuex'
import { 
  Collection, 
  User, 
  Reading, 
  Search, 
  Plus, 
  Edit,
  Delete,
  Refresh,
  DocumentAdd,
  List,
  Grid,
  Calendar,
  Timer
} from '@element-plus/icons-vue'

export default {
  name: 'Books',
  components: {
    Collection,
    User,
    Reading,
    Search,
    Plus,
    Edit,
    Delete,
    Refresh,
    DocumentAdd,
    List,
    Grid,
    Calendar,
    Timer
  },
  data() {
    return {
      dialogVisible: false,
      dialogTitle: '新增书籍',
      bookForm: {
        title: '',
        author: '',
        publisher: '',
        isbn: '',
        description: '',
        coverUrl: '',
        category: '',
        tags: []
      },
      currentBookId: null,
      searchQuery: '',
      authorFilter: '',
      publisherFilter: '',
      sortOption: 'titleAsc',
      currentPage: 1,
      pageSize: 10,
      viewMode: 'table',
      inputTagVisible: false,
      inputTagValue: '',
      planDialogVisible: false,
      planForm: {
        title: '',
        targetMinutes: 30,
        startDate: new Date(),
        endDate: null,
        bookId: null,
        notes: ''
      },
      selectedBookTitle: '',
      // 模拟数据
      mockBooks: [
        { 
          id: 1, 
          title: '三体', 
          author: '刘慈欣', 
          publisher: '重庆出版社', 
          isbn: '9787536692930',
          description: '科幻小说，描述了地球文明与三体文明的接触与博弈',
          coverUrl: '',
          category: 'scifi',
          tags: ['科幻', '雨果奖'],
          createdAt: '2025-03-15T00:00:00Z',
          lastRead: '2025-04-10T00:00:00Z'
        },
        { 
          id: 2, 
          title: '活着', 
          author: '余华', 
          publisher: '作家出版社', 
          isbn: '9787506365437',
          description: '讲述了农村人福贵悲惨的人生遭遇',
          coverUrl: '',
          category: 'literature',
          tags: ['当代文学', '悲情'],
          createdAt: '2025-03-10T00:00:00Z',
          lastRead: '2025-04-05T00:00:00Z'
        },
        { 
          id: 3, 
          title: '百年孤独', 
          author: '加西亚·马尔克斯', 
          publisher: '南海出版公司', 
          isbn: '9787544253994',
          description: '讲述了布恩迪亚家族七代人的传奇故事',
          coverUrl: '',
          category: 'literature',
          tags: ['魔幻现实主义', '诺贝尔文学奖'],
          createdAt: '2025-04-01T00:00:00Z',
          lastRead: null
        },
        { 
          id: 4, 
          title: '1984', 
          author: '乔治·奥威尔', 
          publisher: '北京十月文艺出版社', 
          isbn: '9787530210291',
          description: '描述了一个极权主义社会的恐怖图景',
          coverUrl: '',
          category: 'dystopia',
          tags: ['反乌托邦', '政治寓言'],
          createdAt: '2025-04-12T00:00:00Z',
          lastRead: null
        },
        { 
          id: 5, 
          title: '红楼梦', 
          author: '曹雪芹', 
          publisher: '人民文学出版社', 
          isbn: '9787020002207',
          description: '中国古典四大名著之一，描述了贾、史、王、薛四大家族的兴衰',
          coverUrl: '',
          category: 'classic',
          tags: ['古典文学', '四大名著'],
          createdAt: '2025-02-20T00:00:00Z',
          lastRead: '2025-03-15T00:00:00Z'
        }
      ],
      rules: {
        title: [
          { required: true, message: '请输入书名', trigger: 'blur' }
        ],
        author: [
          { required: true, message: '请输入作者', trigger: 'blur' }
        ]
      },
      categoryOptions: [
        { value: 'literature', label: '文学' },
        { value: 'scifi', label: '科幻' },
        { value: 'biography', label: '传记' },
        { value: 'history', label: '历史' },
        { value: 'business', label: '商业' },
        { value: 'technology', label: '科技' },
        { value: 'selfhelp', label: '自助' },
        { value: 'classic', label: '经典' },
        { value: 'dystopia', label: '反乌托邦' },
        { value: 'other', label: '其他' }
      ]
    }
  },
  computed: {
    ...mapState({
      storeBooks: state => state.books || []
    }),
    books() {
      return this.storeBooks.length > 0 ? this.storeBooks : this.mockBooks;
    },
    filteredBooks() {
      let result = [...this.books];
      
      // 搜索
      if (this.searchQuery) {
        const query = this.searchQuery.toLowerCase();
        result = result.filter(book => 
          book.title.toLowerCase().includes(query) ||
          book.author.toLowerCase().includes(query) ||
          book.publisher.toLowerCase().includes(query) ||
          (book.isbn && book.isbn.toLowerCase().includes(query))
        );
      }
      
      // 作者筛选
      if (this.authorFilter) {
        result = result.filter(book => book.author === this.authorFilter);
      }
      
      // 出版社筛选
      if (this.publisherFilter) {
        result = result.filter(book => book.publisher === this.publisherFilter);
      }
      
      // 排序
      switch (this.sortOption) {
        case 'titleAsc':
          result.sort((a, b) => a.title.localeCompare(b.title));
          break;
        case 'titleDesc':
          result.sort((a, b) => b.title.localeCompare(a.title));
          break;
        case 'dateDesc':
          result.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
          break;
        case 'dateAsc':
          result.sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt));
          break;
      }
      
      return result;
    },
    paginatedBooks() {
      const start = (this.currentPage - 1) * this.pageSize;
      const end = start + this.pageSize;
      return this.filteredBooks.slice(start, end);
    },
    authorOptions() {
      // 获取不重复的作者列表
      return [...new Set(this.books.map(book => book.author))];
    },
    publisherOptions() {
      // 获取不重复的出版社列表
      return [...new Set(this.books.map(book => book.publisher))];
    },
    totalBooks() {
      return this.books.length;
    },
    uniqueAuthors() {
      return this.authorOptions.length;
    },
    recentlyAdded() {
      // 计算最近30天添加的书籍数量
      const thirtyDaysAgo = new Date();
      thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
      
      return this.books.filter(book => {
        const createdDate = new Date(book.createdAt);
        return createdDate >= thirtyDaysAgo;
      }).length;
    }
  },
  methods: {
    ...mapActions(['fetchBook', 'updateBook', 'deleteBook']),
    handleSearch() {
      this.currentPage = 1;
    },
    handleFilter() {
      this.currentPage = 1;
    },
    handleSort() {
      this.currentPage = 1;
    },
    resetFilters() {
      this.searchQuery = '';
      this.authorFilter = '';
      this.publisherFilter = '';
      this.sortOption = 'titleAsc';
      this.currentPage = 1;
    },
    handleSizeChange(size) {
      this.pageSize = size;
    },
    handleCurrentChange(page) {
      this.currentPage = page;
    },
    tableRowClassName({row, rowIndex}) {
      return '';
    },
    handleAdd() {
      this.dialogTitle = '新增书籍'
      this.bookForm = {
        title: '',
        author: '',
        publisher: '',
        isbn: '',
        description: '',
        coverUrl: '',
        category: '',
        tags: []
      }
      this.currentBookId = null
      this.dialogVisible = true
    },
    async handleEdit(book) {
      this.dialogTitle = '编辑书籍'
      this.bookForm = { ...book }
      this.currentBookId = book.id
      this.dialogVisible = true
    },
    async handleDelete(bookId) {
      try {
        await this.$confirm('确认删除该书籍吗？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        
        if (this.storeBooks.length === 0) {
          const index = this.mockBooks.findIndex(book => book.id === bookId);
          if (index !== -1) {
            this.mockBooks.splice(index, 1);
          }
        } else {
          await this.deleteBook(bookId);
        }
        
        this.$message.success('删除成功')
        this.animateStats();
      } catch (error) {
        if (error !== 'cancel') {
          this.$message.error('删除失败')
        }
      }
    },
    async handleSubmit() {
      try {
        if (this.storeBooks.length === 0) {
          if (this.currentBookId) {
            const index = this.mockBooks.findIndex(book => book.id === this.currentBookId);
            if (index !== -1) {
              this.mockBooks[index] = { 
                ...this.bookForm, 
                id: this.currentBookId,
                createdAt: this.mockBooks[index].createdAt
              };
            }
          } else {
            const newId = this.mockBooks.length > 0 
              ? Math.max(...this.mockBooks.map(book => book.id)) + 1 
              : 1;
            
            this.mockBooks.push({
              ...this.bookForm,
              id: newId,
              createdAt: new Date().toISOString(),
              lastRead: null
            });
          }
          
          this.dialogVisible = false;
          this.$message.success('保存成功');
          this.animateStats();
        } else {
          if (this.currentBookId) {
            await this.updateBook({
              bookId: this.currentBookId,
              data: this.bookForm
            });
          }
          this.dialogVisible = false;
          this.$message.success('保存成功');
        }
      } catch (error) {
        this.$message.error('保存失败');
      }
    },
    createReadingPlan(book) {
      this.selectedBookTitle = book.title;
      this.planForm = {
        title: `《${book.title}》阅读计划`,
        targetMinutes: 30,
        startDate: new Date(),
        endDate: this.getDefaultEndDate(),
        bookId: book.id,
        notes: ''
      };
      this.planDialogVisible = true;
    },
    async savePlan() {
      this.$message.success('阅读计划创建成功');
      this.planDialogVisible = false;
    },
    getDefaultEndDate() {
      const date = new Date();
      date.setDate(date.getDate() + 30);
      return date;
    },
    formatDate(dateString) {
      if (!dateString) return '暂无记录';
      
      const date = new Date(dateString);
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, '0');
      const day = String(date.getDate()).padStart(2, '0');
      
      return `${year}-${month}-${day}`;
    },
    getRandomColor(id) {
      const colors = [
        '#409EFF', '#67C23A', '#E6A23C', '#F56C6C', '#909399',
        '#8e44ad', '#3498db', '#1abc9c', '#e74c3c', '#2c3e50'
      ];
      return colors[id % colors.length];
    },
    beforeCoverUpload(file) {
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = () => {
        this.bookForm.coverUrl = reader.result;
      };
      return false;
    },
    showTagInput() {
      this.inputTagVisible = true;
      this.$nextTick(() => {
        this.$refs.tagInputRef.focus();
      });
    },
    handleAddTag() {
      if (this.inputTagValue) {
        if (!this.bookForm.tags.includes(this.inputTagValue)) {
          this.bookForm.tags.push(this.inputTagValue);
        }
      }
      this.inputTagVisible = false;
      this.inputTagValue = '';
    },
    handleRemoveTag(tag) {
      this.bookForm.tags.splice(this.bookForm.tags.indexOf(tag), 1);
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
    this.fetchBook && this.fetchBook();
    
    setTimeout(() => {
      this.animateStats();
    }, 500);
  }
}
</script>

<style scoped>
.books-container {
  padding: 20px;
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

.stat-icon.blue {
  background: linear-gradient(135deg, #3498db, #2980b9);
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

.main-card {
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.title {
  font-size: 18px;
  font-weight: bold;
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

.search-filter-container {
  margin-bottom: 20px;
  padding: 15px;
  background-color: #f5f7fa;
  border-radius: 8px;
}

.search-box {
  margin-bottom: 15px;
}

.filter-section {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.view-toggle {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 15px;
}

.empty-books {
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

.book-expand {
  display: flex;
  padding: 20px;
  gap: 20px;
}

.book-cover {
  width: 120px;
  height: 180px;
  overflow: hidden;
  border-radius: 4px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.book-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.book-details {
  flex: 1;
}

.book-title-cell {
  display: flex;
  align-items: center;
}

.book-title {
  font-weight: bold;
}

.book-description {
  color: #606266;
  margin: 10px 0 15px;
}

.book-meta {
  display: flex;
  gap: 20px;
  color: #909399;
  font-size: 14px;
}

.book-meta span {
  display: flex;
  align-items: center;
  gap: 5px;
}

.action-buttons {
  display: flex;
  gap: 8px;
}

.action-buttons .el-button {
  transition: all 0.2s ease;
}

.action-buttons .el-button:hover {
  transform: scale(1.1);
}

.book-cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  margin-bottom: 20px;
}

.book-card {
  border-radius: 8px;
  overflow: hidden;
  transition: all 0.3s ease;
  height: 100%;
}

.book-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}

.book-card-cover {
  height: 150px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.book-card-title {
  font-size: 60px;
  font-weight: bold;
  color: white;
}

.book-card-content {
  padding: 15px;
}

.book-card-name {
  margin: 0 0 10px;
  font-size: 16px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.book-card-author {
  margin: 0 0 5px;
  color: #606266;
  font-size: 14px;
}

.book-card-publisher {
  margin: 0 0 15px;
  color: #909399;
  font-size: 13px;
}

.book-card-actions {
  display: flex;
  justify-content: space-around;
}

.pagination-container {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.avatar-uploader {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  width: 120px;
  height: 180px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.avatar-uploader:hover {
  border-color: #409EFF;
}

.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
}

.avatar {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.upload-tip {
  font-size: 12px;
  color: #909399;
  margin-top: 8px;
}

.book-tag {
  margin-right: 8px;
  margin-bottom: 8px;
}

.tag-input {
  width: 120px;
  margin-right: 8px;
  vertical-align: top;
}

.button-new-tag {
  margin-bottom: 8px;
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

/* 权限管理样式 */
.role-badge {
  margin-left: 5px;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .stats-row {
    flex-direction: column;
  }
  
  .filter-section {
    flex-direction: column;
  }
  
  .book-expand {
    flex-direction: column;
    align-items: center;
  }
  
  .book-meta {
    flex-direction: column;
    gap: 10px;
  }
  
  .book-cards {
    grid-template-columns: 1fr;
  }
}
</style>
