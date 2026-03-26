<template>
    <div>
        <div class="position-relative">
            <section class="section-shaped my-0">
                <div class="shape shape-style-1 shape-default shape-skew">
                    <span></span>
                    <span></span>
                    <span></span>
                    <span></span>
                    <span></span>
                    <span></span>
                    <span></span>
                    <span></span>
                    <span></span>
                </div>
                <div class="container shape-container d-flex">
                    <div class="col px-0">
                        <div class="row">
                            <div class="col-lg-8">
                                <h1 class="display-3 text-white">試卷分析系統
                                    <span>AI輔助評估與自動化評分平台</span>
                                </h1>
                                <p class="lead text-white">支援 PDF 與 Word 檔</p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
        </div>

        <section class="section section-lg pt-lg-0 mt--200">
            <div class="container" id="upload-area">
                <div class="row row-grid">
                    <div class="col-lg-4">
                        <card class="border-0" hover shadow body-classes="py-5">
                            <icon name="ni ni-cloud-upload-96" type="primary" rounded class="mb-4"></icon>
                            <h6 class="text-primary text-uppercase">上傳試卷</h6>
                            <p class="description mt-3">可一次選多個 PDF 或 Word 檔案。</p>
                            <base-button tag="button" @click="openFilePicker" type="primary" class="mt-3">
                                上傳文件
                            </base-button>
                        </card>
                    </div>

                    <div class="col-lg-4">
                        <card class="border-0" hover shadow body-classes="py-5">
                            <icon name="ni ni-bullet-list-67" type="success" rounded class="mb-4"></icon>
                            <h6 class="text-success text-uppercase">選擇科目</h6>
                            <p class="description mt-3">先選擇本次試卷對應科目。</p>
                            <select class="form-control" v-model="selectedSubject">
                                <option disabled value="">請選擇科目</option>
                                <option>國文</option>
                                <option>英文</option>
                                <option>數學</option>
                                <option>自然</option>
                                <option>社會</option>
                            </select>
                        </card>
                    </div>

                    <div class="col-lg-4">
                        <card class="border-0" hover shadow body-classes="py-5">
                            <icon name="ni ni-chart-bar-32" type="warning" rounded class="mb-4"></icon>
                            <h6 class="text-warning text-uppercase">查看分析</h6>
                            <p class="description mt-3">目前先顯示已讀取檔案，後續可接分析流程。</p>
                        <base-button tag="button" type="warning" class="mt-3" @click="generateAnalysis">
                                查看結果
                            </base-button>
                        </card>
                    </div>
                </div>

                <div class="row mt-4" id="files-preview" v-if="uploadedFiles.length > 0">
                    <div class="col-lg-12">
                        <card class="border-0" shadow body-classes="p-4">
                            <h4 class="mb-3">已讀取檔案</h4>
                            <p v-if="selectedSubject" class="text-muted mb-3">目前科目：{{ selectedSubject }}</p>
                            <p v-else class="text-muted mb-3">目前科目：尚未選擇</p>

                      <div class="row">
                                <div class="col-md-6 col-lg-4 mb-4" v-for="file in uploadedFiles" :key="file.id">
                                    <div class="preview-card p-3 h-100 border rounded">
                                        <h6 class="mb-1 text-truncate" :title="file.name">{{ file.name }}</h6>
                                        <small class="text-muted d-block mb-2">{{ formatFileSize(file.size) }}</small>

                                        <div v-if="file.kind === 'pdf'">
                                            <embed :src="file.previewUrl" type="application/pdf" class="pdf-preview" />
                                        </div>

                                        <div v-else-if="file.kind === 'word'" class="word-preview border rounded p-3">
                                          <div class="font-weight-bold">Word 文件已讀取</div>
                                          <small class="text-muted">目前先顯示檔案資訊，可於下一步接入內容解析。</small>
                                        </div>

                                        <div v-else class="text-muted">不支援的檔案格式</div>
                                    </div>
                                </div>
                            </div>
                        </card>
                    </div>
                </div>

                  <div class="row mt-4" id="analysis-result" v-if="analysisResult">
                    <div class="col-lg-12">
                      <card class="border-0" shadow body-classes="p-4">
                        <div class="d-flex justify-content-between align-items-center mb-3 flex-wrap">
                          <h4 class="mb-0">評分結果</h4>
                          <span class="badge badge-primary badge-pill px-3 py-2">總分 {{ analysisResult.total }}/100</span>
                        </div>

                        <div class="row mb-3">
                          <div class="col-md-6 col-lg-3 mb-3" v-for="item in analysisResult.items" :key="item.key">
                            <div class="score-item p-3 border rounded h-100">
                              <small class="text-muted d-block mb-1">{{ item.label }}（{{ item.weightLabel }}）</small>
                              <div class="h5 mb-1">原始分數：{{ item.score }} / 100</div>
                              <small class="text-muted">加權得分：{{ item.weightedScore }}</small>
                            </div>
                          </div>
                        </div>

                        <h5 class="mb-2">建議</h5>
                        <ul class="mb-0 pl-3">
                          <li v-for="(tip, index) in analysisResult.suggestions" :key="`${index}-${tip}`" class="mb-1">
                            {{ tip }}
                          </li>
                        </ul>
                      </card>
                    </div>
                  </div>
            </div>
        </section>

        <input
            ref="fileInput"
            type="file"
          accept="application/pdf,.doc,.docx,application/msword,application/vnd.openxmlformats-officedocument.wordprocessingml.document"
            multiple
            class="d-none"
            @change="handleFileChange"
        />
    </div>
</template>

<script>
export default {
  name: "landing",
  data() {
    return {
      selectedSubject: "",
      uploadedFiles: [],
      analysisResult: null
    };
  },
  methods: {
    openFilePicker() {
      this.$refs.fileInput.click();
    },
    handleFileChange(event) {
      const files = Array.from(event.target.files || []);

      // Release previous object URLs before replacing previews.
      this.cleanupPreviewUrls();

      this.uploadedFiles = files.map((file, index) => {
        const isPdf = file.type === "application/pdf";
        const isWord = file.type === "application/msword" || file.type === "application/vnd.openxmlformats-officedocument.wordprocessingml.document" || /\.(doc|docx)$/i.test(file.name);
        const previewUrl = URL.createObjectURL(file);

        return {
          id: `${file.name}-${index}`,
          name: file.name,
          size: file.size,
          kind: isPdf ? "pdf" : isWord ? "word" : "other",
          previewUrl
        };
      });
    },
    cleanupPreviewUrls() {
      this.uploadedFiles.forEach(file => {
        if (file.previewUrl) {
          URL.revokeObjectURL(file.previewUrl);
        }
      });
    },
    formatFileSize(size) {
      if (size < 1024) {
        return `${size} B`;
      }

      if (size < 1024 * 1024) {
        return `${(size / 1024).toFixed(1)} KB`;
      }

      return `${(size / (1024 * 1024)).toFixed(2)} MB`;
    },
    generateAnalysis() {
      const fileCount = this.uploadedFiles.length;
      const hasSubject = this.selectedSubject !== "";

      if (!fileCount) {
        alert("請先上傳至少一份試卷檔案。");
        return;
      }

      if (!hasSubject) {
        alert("請先選擇科目。");
        return;
      }

      const base = Math.min(88, 62 + fileCount * 4);
      const subjectBonus = this.selectedSubject === "數學" || this.selectedSubject === "英文" ? 3 : 1;

      const items = [
        { key: "accuracy", label: "內容正確性", weight: 0.2, weightLabel: "20%", score: this.clamp(base + 8 + subjectBonus, 0, 100) },
        { key: "logic", label: "組織邏輯", weight: 0.25, weightLabel: "25%", score: this.clamp(base + 4, 0, 100) },
        { key: "critical", label: "個人批判性見解", weight: 0.25, weightLabel: "25%", score: this.clamp(base + 2, 0, 100) },
        { key: "language", label: "語言表達", weight: 0.1, weightLabel: "10%", score: this.clamp(base - 3, 0, 100) },
        { key: "creativity", label: "創意與視覺呈現", weight: 0.1, weightLabel: "10%", score: this.clamp(base - 2, 0, 100) },
        { key: "citation", label: "參考文獻與 AI 使用聲明", weight: 0.1, weightLabel: "10%", score: this.clamp(base - 1, 0, 100) }
      ];

      const itemsWithWeightedScore = items.map(item => ({
        ...item,
        weightedScore: (item.score * item.weight).toFixed(1)
      }));

      const total = Math.round(itemsWithWeightedScore.reduce((sum, item) => sum + Number(item.weightedScore), 0));

      this.analysisResult = {
        total,
        items: itemsWithWeightedScore,
        suggestions: this.buildSuggestions(total)
      };

      this.scrollToSection("analysis-result");
    },
    buildSuggestions(total) {
      const tips = [];

      if (total >= 85) {
        tips.push("整體表現優秀，可開始加強進階題型與跨單元整合。", "建議每週進行一次限時模擬，維持應試節奏。", "保留錯題整理習慣，避免重複失分。");
      } else if (total >= 70) {
        tips.push("基礎掌握穩定，建議優先加強中高難度題目。", "針對錯題類型建立複習清單，集中突破薄弱章節。", "可透過分段計時練習提升作答效率。" );
      } else {
        tips.push("先回到核心觀念與基本題型，建立穩定得分能力。", "建議將試卷分成小單元練習，降低一次性學習負擔。", "每次練習後做 5 分鐘錯因回顧，確認是觀念或粗心問題。");
      }

      return tips;
    },
    clamp(value, min, max) {
      return Math.max(min, Math.min(max, value));
    },
    scrollToSection(id) {
      const target = document.getElementById(id);
      if (target) {
        target.scrollIntoView({ behavior: "smooth", block: "start" });
      }
    }
  },
  beforeDestroy() {
    this.cleanupPreviewUrls();
  }
};
</script>

<style scoped>
.preview-card {
  background: #fff;
}

.pdf-preview {
  width: 100%;
  height: 220px;
  border: 1px solid #e9ecef;
  border-radius: 0.25rem;
}

.score-item {
  background: #f8fbff;
}

.word-preview {
  background: #f8fbff;
}
</style>
