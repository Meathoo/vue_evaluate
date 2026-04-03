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
                  <div class="col-lg-12">
                        <card class="border-0" hover shadow body-classes="py-5">
                            <icon name="ni ni-cloud-upload-96" type="primary" rounded class="mb-4"></icon>
                            <h6 class="text-primary text-uppercase">上傳試卷</h6>
                            <p class="description mt-3">可一次選多個 PDF 或 Word 檔案。</p>
                            <div class="d-flex flex-wrap action-button-row mt-3">
                              <base-button tag="button" @click="openFilePicker" type="primary" class="mr-2 mb-2" :disabled="showResultSection">
                                上傳文件
                              </base-button>
                            </div>
                        </card>
                    </div>

                    <!-- <div class="col-lg-4">
                        <card class="border-0" hover shadow body-classes="py-5">
                            <icon name="ni ni-bullet-list-67" type="success" rounded class="mb-4"></icon>
                      <h6 class="text-success text-uppercase">逐檔輸入 Prompt</h6>
                      <p class="description mt-3">上傳後可針對每份檔案填寫分析重點、評分需求與輸出格式。</p>
                      <small class="text-muted">例如：請先摘要重點，再依六項指標逐項評分並給建議。</small>
                        </card>
                    </div>  -->

                </div>

                <div class="row mt-4" id="files-preview" v-if="uploadedFiles.length > 0">
                    <div class="col-lg-12">
                        <card class="border-0" shadow body-classes="p-4">
                            <h4 class="mb-3">已讀取檔案</h4>
                      <p class="text-muted mb-3">請為每份檔案填寫評分標準，系統會依各檔案需求分析。</p>

                      <transition name="prompt-banner">
                        <div v-if="promptWarning" class="prompt-warning mb-3" role="status" aria-live="polite">
                          <i class="ni ni-bell-55 mr-2"></i>
                          <span>{{ promptWarning }}</span>
                        </div>
                      </transition>

                      <div class="row">
                                <div class="col-md-6 col-lg-4 mb-4" v-for="file in uploadedFiles" :key="file.id">
                              <div
                                class="preview-card p-3 h-100 border rounded"
                                :class="{ 'preview-card--active': selectedFileId === file.id }"
                                @click="selectFile(file)"
                                role="button"
                                tabindex="0"
                              >
                                        <h6 class="mb-1 text-truncate" :title="file.name">{{ file.name }}</h6>
                                        <small class="text-muted d-block mb-2">{{ formatFileSize(file.size) }}</small>
                                <small class="text-primary d-block mb-3">點擊此檔案即可查看個別結果</small>

                                        <div v-if="file.kind === 'pdf'">
                                            <embed :src="file.previewUrl" type="application/pdf" class="pdf-preview" />
                                        </div>

                                        <div v-else-if="file.kind === 'word'" class="word-preview border rounded p-3">
                                          <div class="font-weight-bold">Word 文件已讀取</div>
                                          <small class="text-muted">目前先顯示檔案資訊，可於下一步接入內容解析。</small>
                                        </div>

                                        <div v-else class="text-muted">不支援的檔案格式</div>

                    <div class="mt-3">
                      <template v-if="showResultSection">
                        <label class="mb-1 font-weight-bold">評分標準</label>
                        <p class="mb-0 prompt-summary-line" :title="file.prompt || '尚未填寫評分標準'">{{ file.prompt || '尚未填寫評分標準' }}</p>
                      </template>
                      <template v-else>
                        <label class="mb-1 font-weight-bold">評分標準</label>
                        <textarea
                          class="form-control"
                          rows="4"
                          v-model="file.prompt"
                          placeholder="請填寫評分標準"
                          @click.stop
                          :disabled="showResultSection"
                        ></textarea>
                        <small class="text-muted d-block mt-1">已輸入 {{ file.prompt.length }} 字</small>
                      </template>
                    </div>
                                    </div>
                                </div>
                            </div>

                              <div class="text-right mt-3 d-flex justify-content-end">
                                <base-button tag="button" type="warning" class="mr-2" :disabled="!canSubmitResults" @click="showResults">
                                  前往結果
                                </base-button>
                                <base-button tag="button" type="primary" :disabled="uploadedFiles.length === 0" @click="clearUploadedFiles">
                                  清除檔案
                                </base-button>
                              </div>
                        </card>
                    </div>
                </div>

                  <div class="row mt-4" id="analysis-result" v-if="showResultSection">
                    <div class="col-lg-12">
                      <card class="border-0" shadow body-classes="p-4">
                        <template v-if="selectedFile && analysisResult">
                          <div class="d-flex justify-content-between align-items-center mb-3 flex-wrap">
                            <div>
                              <h4 class="mb-1">評分結果</h4>
                              <small class="text-muted">目前檔案：{{ selectedFile.name }}</small>
                            </div>
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
                        </template>

                        <div v-else class="text-center py-5">
                          <h4 class="mb-2">結果區已建立</h4>
                          <p class="text-muted mb-0">請先點選上方任一份已上傳檔案，系統就會顯示該檔案的分數與建議。</p>
                        </div>
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
      uploadedFiles: [],
      selectedFileId: "",
      analysisResult: null,
      showResultSection: false,
      promptWarning: "",
      promptWarningTimer: null
    };
  },
  computed: {
    selectedFile() {
      return this.uploadedFiles.find(file => file.id === this.selectedFileId) || null;
    },
    canSubmitResults() {
      return this.uploadedFiles.length > 0 && this.uploadedFiles.every(file => file.prompt.trim().length > 0);
    }
  },
  methods: {
    openFilePicker() {
      if (this.showResultSection) {
        this.triggerPromptWarning("已送出結果後無法再新增檔案，請先清除檔案後重新上傳。");
        return;
      }

      this.$refs.fileInput.click();
    },
    clearUploadedFiles() {
      this.clearPromptWarning();
      this.cleanupPreviewUrls();
      this.uploadedFiles = [];
      this.selectedFileId = "";
      this.analysisResult = null;
      this.showResultSection = false;

      if (this.$refs.fileInput) {
        this.$refs.fileInput.value = "";
      }
    },
    handleFileChange(event) {
      const files = Array.from(event.target.files || []);
      this.clearPromptWarning();
      const existingNames = new Set(this.uploadedFiles.map(file => file.name));
      const duplicateNames = [];

      const newUploadedFiles = files.map((file, index) => {
        const isPdf = file.type === "application/pdf";
        const isWord = file.type === "application/msword" || file.type === "application/vnd.openxmlformats-officedocument.wordprocessingml.document" || /\.(doc|docx)$/i.test(file.name);
        const previewUrl = URL.createObjectURL(file);

        if (existingNames.has(file.name)) {
          duplicateNames.push(file.name);
          return null;
        }

        existingNames.add(file.name);

        return {
          id: `${file.name}-${Date.now()}-${index}`,
          name: file.name,
          size: file.size,
          kind: isPdf ? "pdf" : isWord ? "word" : "other",
          prompt: "",
          previewUrl
        };
      }).filter(Boolean);

      this.uploadedFiles = [...this.uploadedFiles, ...newUploadedFiles];

      if (duplicateNames.length > 0) {
        this.triggerPromptWarning(`已略過重複檔名：${duplicateNames.join("、")}`);
      }

      if (this.$refs.fileInput) {
        this.$refs.fileInput.value = "";
      }
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
    selectFile(file) {
      if (!file.prompt.trim()) {
        this.triggerPromptWarning("請先為此檔案填寫評分標準，再查看個別結果。");
        return;
      }

      this.selectedFileId = file.id;
      this.analysisResult = this.buildFileAnalysis(file);
    },
    showResults() {
      if (!this.canSubmitResults) {
        this.triggerPromptWarning("請先為所有已上傳檔案填寫評分標準，才能提交結果。");
        return;
      }

      if (this.selectedFile && this.analysisResult) {
        this.showResultSection = true;
        this.scrollToSection("analysis-result");
        return;
      }

      const firstReadyFile = this.uploadedFiles.find(file => file.prompt.trim().length > 0);

      if (!firstReadyFile) {
        this.triggerPromptWarning("請先為至少一份檔案填寫評分標準，才能前往結果。");
        return;
      }

      this.selectedFileId = firstReadyFile.id;
      this.analysisResult = this.buildFileAnalysis(firstReadyFile);
      this.showResultSection = true;
      this.scrollToSection("analysis-result");
    },
    triggerPromptWarning(message) {
      this.promptWarning = message;

      if (this.promptWarningTimer) {
        clearTimeout(this.promptWarningTimer);
      }

      this.promptWarningTimer = setTimeout(() => {
        this.promptWarning = "";
        this.promptWarningTimer = null;
      }, 2600);
    },
    clearPromptWarning() {
      if (this.promptWarningTimer) {
        clearTimeout(this.promptWarningTimer);
        this.promptWarningTimer = null;
      }

      this.promptWarning = "";
    },
    buildFileAnalysis(file) {
      const promptLength = file.prompt.trim().length;

      const base = Math.min(88, 62 + Math.floor(promptLength / 8));
      const promptBonus = promptLength >= 120 ? 4 : promptLength >= 60 ? 2 : 0;

      const items = [
        { key: "accuracy", label: "內容正確性", weight: 0.2, weightLabel: "20%", score: this.clamp(base + 8 + promptBonus, 0, 100) },
        { key: "logic", label: "組織邏輯", weight: 0.25, weightLabel: "25%", score: this.clamp(base + 4 + promptBonus, 0, 100) },
        { key: "critical", label: "個人批判性見解", weight: 0.25, weightLabel: "25%", score: this.clamp(base + 2 + promptBonus, 0, 100) },
        { key: "language", label: "語言表達", weight: 0.1, weightLabel: "10%", score: this.clamp(base - 3 + promptBonus, 0, 100) },
        { key: "creativity", label: "創意與視覺呈現", weight: 0.1, weightLabel: "10%", score: this.clamp(base - 2 + promptBonus, 0, 100) },
        { key: "citation", label: "參考文獻與 AI 使用聲明", weight: 0.1, weightLabel: "10%", score: this.clamp(base - 1 + promptBonus, 0, 100) }
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

      return this.analysisResult;
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
    if (this.promptWarningTimer) {
      clearTimeout(this.promptWarningTimer);
    }
    this.cleanupPreviewUrls();
  }
};
</script>

<style scoped>
.prompt-warning {
  display: flex;
  align-items: center;
  padding: 0.85rem 1rem;
  border-radius: 0.5rem;
  background: linear-gradient(90deg, rgba(255, 236, 209, 0.96), rgba(255, 247, 205, 0.98));
  border: 1px solid rgba(245, 158, 11, 0.35);
  color: #9a5b00;
  font-weight: 600;
  box-shadow: 0 8px 20px rgba(245, 158, 11, 0.12);
}

.prompt-warning i {
  animation: prompt-bell 0.8s ease-in-out 0s 2;
}

.prompt-banner-enter-active {
  animation: prompt-pop 0.22s ease-out;
}

.prompt-banner-leave-active {
  animation: prompt-fade 0.18s ease-in;
}

@keyframes prompt-pop {
  from {
    opacity: 0;
    transform: translateY(-8px) scale(0.98);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes prompt-fade {
  from {
    opacity: 1;
    transform: translateY(0);
  }
  to {
    opacity: 0;
    transform: translateY(-6px);
  }
}

@keyframes prompt-bell {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(-10deg); }
  50% { transform: rotate(10deg); }
  75% { transform: rotate(-6deg); }
}

.preview-card {
  background: #fff;
  transition: transform 0.15s ease, box-shadow 0.2s ease, border-color 0.2s ease;
  cursor: pointer;
}

.preview-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(50, 50, 93, 0.12);
}

.preview-card:active {
  transform: translateY(0);
}

.preview-card--active {
  border-color: #5e72e4 !important;
  box-shadow: 0 0 0 3px rgba(94, 114, 228, 0.2);
}

.prompt-summary-line {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  word-break: normal;
  margin-bottom: 0;
}

.action-button-row .btn {
  min-width: 132px;
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
