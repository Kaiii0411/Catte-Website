<template>
  <div class="app-wrapper">

    <!-- ── Header ── -->
    <header class="app-header">
      <div class="header-inner">
        <div class="header-brand">
          <h1 class="brand-title">Simple Tracker</h1>
        </div>
      </div>
    </header>

    <!-- ── Main ── -->
    <main class="app-main">
      <el-tabs v-model="activeTab" class="catte-tabs">

        <!-- ── Tab: Members ── -->
        <el-tab-pane label="Members" name="members">
          <div class="tab-content">
            <div class="table-wrap">
              <el-table :data="names" class="catte-table" border>
                <el-table-column label="Name" prop="name" />
                <el-table-column label="Status" width="110">
                  <template #default="scope">
                    <el-tag :type="scope.row.inRound ? 'success' : 'danger'" class="status-tag">
                      {{ scope.row.inRound ? '✓ In' : '✗ Out' }}
                    </el-tag>
                  </template>
                </el-table-column>
                <el-table-column width="110" align="center">
                  <template #default="scope">
                    <el-button size="small" type="success" class="action-btn"
                      @click="addToRound(scope.row.id, scope.row.name)">+</el-button>
                    <el-button size="small" type="danger" class="action-btn"
                      @click="removeFromRound(scope.row.id, scope.row.name)">−</el-button>
                  </template>
                </el-table-column>
              </el-table>
            </div>

            <div class="reset-area">
              <button class="reset-btn" @click="resetRound()">
                🔄 Reset Round
              </button>
            </div>
          </div>
        </el-tab-pane>

        <!-- ── Tab: Main ── -->
        <el-tab-pane label="Main" name="main">
          <div class="tab-content">

            <el-collapse class="add-collapse">
              <el-collapse-item title="➕  Add Custom Member">
                <div class="add-form-wrap">
                  <el-form :model="member" label-width="auto" style="max-width: 480px">
                    <el-form-item label="Name">
                      <el-input v-model="member.name" placeholder="Enter name…" />
                    </el-form-item>
                    <el-form-item>
                      <el-button type="primary" @click="addMember()">Add</el-button>
                    </el-form-item>
                  </el-form>
                </div>
              </el-collapse-item>
            </el-collapse>

            <div v-if="members.length === 0" class="empty-state">
              <p>🃏 No members in this round yet.</p>
              <p class="empty-hint">Go to the <b>Members</b> tab and press <b>+</b> to add some!</p>
            </div>

            <div v-else class="cards-grid">
              <div v-for="mem in members" :key="mem.id" class="member-card">
                <div class="card-glow"></div>
                <div class="card-name">{{ mem.name }}</div>
                <div class="card-coins">
                  <span class="coins-label">Coins</span>
                  <span class="coins-value">{{ mem.coins }}</span>
                </div>
                <div class="card-actions">
                  <el-button type="success" size="small" class="coin-btn" @click="addCoins(mem.id, 5)">+5</el-button>
                  <el-button type="success" size="small" class="coin-btn" @click="addCoins(mem.id, 10)">+10</el-button>
                  <el-button type="danger"  size="small" class="coin-btn" @click="addCoins(mem.id, -5)">−5</el-button>
                  <el-button type="danger"  size="small" class="coin-btn" @click="addCoins(mem.id, -10)">−10</el-button>
                </div>
              </div>
            </div>

          </div>
        </el-tab-pane>

        <!-- ── Tab: Logs ── -->
        <el-tab-pane label="Logs" name="logs">
          <div class="tab-content">
            <div v-if="logs.length === 0" class="empty-state">
              <p>📋 No activity logged yet.</p>
            </div>
            <div v-else class="table-wrap">
              <el-table :data="logs" class="catte-table" border>
                <el-table-column label="#" prop="index" width="56" align="center" />
                <el-table-column label="Name" prop="name" />
                <el-table-column label="Coins" width="120" align="center">
                  <template #default="{ row }">
                    <span :class="row.coins < 0 ? 'coins-neg' : 'coins-pos'">
                      {{ row.coins > 0 ? '▲ +' + row.coins : '▼ ' + row.coins }}
                    </span>
                  </template>
                </el-table-column>
                <el-table-column label="Time" prop="created" />
              </el-table>
            </div>
          </div>
        </el-tab-pane>

        <!-- ── Tab: Total ── -->
        <el-tab-pane label="Total" name="total">
          <div class="tab-content">
            <div v-if="members.length === 0" class="empty-state">
              <p>📊 No data yet.</p>
            </div>
            <div v-else class="table-wrap">
              <el-table :data="sortedMembers" class="catte-table" border>
                <el-table-column label="Rank" width="70" align="center">
                  <template #default="scope">
                    <span class="rank-badge">
                      <template v-if="scope.$index === 0">🥇</template>
                      <template v-else-if="scope.$index === 1">🥈</template>
                      <template v-else-if="scope.$index === 2">🥉</template>
                      <template v-else>{{ scope.$index + 1 }}</template>
                    </span>
                  </template>
                </el-table-column>
                <el-table-column label="Name" prop="name" />
                <el-table-column label="Coins" width="120" align="center">
                  <template #default="scope">
                    <span class="coins-value-sm">{{ scope.row.coins + 10 }}</span>
                  </template>
                </el-table-column>
              </el-table>
            </div>
          </div>
        </el-tab-pane>

      </el-tabs>
    </main>

  </div>
</template>

<style scoped>
/* ── Layout ── */
.app-wrapper {
  min-height: 100vh;
  background: linear-gradient(160deg, #0d1117 0%, #12192b 55%, #0d1117 100%);
  font-family: 'Outfit', sans-serif;
}

/* ── Header ── */
.app-header {
  background: linear-gradient(135deg,
    rgba(99, 102, 241, 0.18) 0%,
    rgba(168, 85, 247, 0.18) 100%);
  border-bottom: 1px solid rgba(255, 255, 255, 0.07);
  padding: 20px 32px;
  backdrop-filter: blur(12px);
}

.header-inner {
  max-width: 960px;
  margin: 0 auto;
}

.header-brand {
  display: flex;
  align-items: center;
  gap: 14px;
}

.brand-icon {
  font-size: 2.2rem;
  filter: drop-shadow(0 0 12px rgba(99, 102, 241, 0.7));
}

.brand-title {
  font-size: 1.9rem;
  font-weight: 800;
  background: linear-gradient(135deg, #818cf8 0%, #c084fc 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  letter-spacing: -0.5px;
}

/* ── Main Content ── */
.app-main {
  max-width: 960px;
  margin: 0 auto;
  padding: 28px 20px 60px;
}

/* ── Tabs ── */
.catte-tabs {
  --el-tabs-header-height: 44px;
}

/* ── Tab Content ── */
.tab-content {
  padding: 20px 0;
}

/* ── Tables ── */
.table-wrap {
  display: flex;
  justify-content: center;
}

.catte-table {
  width: 100%;
  max-width: 680px;
  border-radius: 12px;
  overflow: hidden;
}

/* ── Status Tag ── */
.status-tag {
  font-weight: 600;
  font-size: 0.78rem;
  letter-spacing: 0.3px;
}

/* ── Action Buttons ── */
.action-btn {
  font-weight: 700;
}

/* ── Reset Button ── */
.reset-area {
  display: flex;
  justify-content: center;
  margin-top: 24px;
}

.reset-btn {
  background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);
  border: none;
  color: white;
  font-family: 'Outfit', sans-serif;
  font-size: 1rem;
  font-weight: 700;
  padding: 12px 36px;
  border-radius: 12px;
  cursor: pointer;
  letter-spacing: 0.3px;
  box-shadow: 0 4px 20px rgba(245, 158, 11, 0.3);
  transition: all 0.25s ease;
}

.reset-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 32px rgba(245, 158, 11, 0.5);
}

.reset-btn:active {
  transform: translateY(0);
}

/* ── Add Collapse ── */
.add-collapse {
  margin-bottom: 28px;
  border-radius: 12px;
  overflow: hidden;
}

.add-form-wrap {
  padding: 16px 8px;
}

/* ── Empty State ── */
.empty-state {
  text-align: center;
  padding: 64px 20px;
  color: #475569;
}

.empty-state p {
  font-size: 1.1rem;
  margin-bottom: 8px;
}

.empty-hint {
  font-size: 0.9rem !important;
  color: #334155 !important;
}

/* ── Member Cards Grid ── */
.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 20px;
  padding: 4px 0;
}

.member-card {
  position: relative;
  background: linear-gradient(145deg,
    rgba(255, 255, 255, 0.055) 0%,
    rgba(255, 255, 255, 0.02) 100%);
  border: 1px solid rgba(255, 255, 255, 0.09);
  border-radius: 18px;
  padding: 24px 20px 20px;
  backdrop-filter: blur(14px);
  overflow: hidden;
  transition: transform 0.3s ease, box-shadow 0.3s ease, border-color 0.3s ease;
}

.card-glow {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, #818cf8, #c084fc);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.member-card:hover {
  transform: translateY(-5px);
  border-color: rgba(129, 140, 248, 0.35);
  box-shadow: 0 18px 48px rgba(0, 0, 0, 0.45),
              0 0 0 1px rgba(129, 140, 248, 0.1);
}

.member-card:hover .card-glow {
  opacity: 1;
}

.card-name {
  font-size: 1.15rem;
  font-weight: 700;
  color: #e2e8f0;
  margin-bottom: 16px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.card-coins {
  display: flex;
  align-items: baseline;
  gap: 10px;
  margin-bottom: 20px;
}

.coins-label {
  font-size: 0.78rem;
  color: #64748b;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.8px;
}

.coins-value {
  font-size: 2.8rem;
  font-weight: 800;
  color: #f0a500;
  line-height: 1;
  text-shadow: 0 0 24px rgba(240, 165, 0, 0.45);
}

.card-actions {
  display: flex;
  gap: 6px;
}

.coin-btn {
  flex: 1;
  font-weight: 700 !important;
  font-family: 'Outfit', sans-serif !important;
}

/* ── Logs ── */
.coins-pos {
  color: #4ade80;
  font-weight: 700;
  font-size: 0.95rem;
}

.coins-neg {
  color: #f87171;
  font-weight: 700;
  font-size: 0.95rem;
}

/* ── Total / Rank ── */
.rank-badge {
  font-size: 1.15rem;
}

.coins-value-sm {
  font-weight: 700;
  font-size: 1rem;
  color: #f0a500;
}
</style>

<script type="text/javascript">
    export default {
        name: 'LogCoins',
        data() {
            return {
                activeTab: 'members',
                members: [],
                member: {
                    id: 0,
                    name: '',
                    coins: 0,
                    nameId: ''
                },
                logs: [],
                names: [
                    { id: "N1", name: "Nghĩa 😎", inRound: false},
                    { id: "T2", name: "C Tiền 🌸", inRound: false },
                    { id: "H3", name: "C Hương 🌻", inRound: false },
                    { id: "L4", name: "C Linh 🌷", inRound: false },
                    { id: "H5", name: "A Hiếu", inRound: false },
                    { id: "K6", name: "A Kiệt", inRound: false },
                    { id: "T7", name: "A Tân", inRound: false },
                    { id: "V8", name: "A Vinh", inRound: false },
                    { id: "V9", name: "A Vương", inRound: false },
                    { id: "T10", name: "C Thy🌼", inRound: false },
                    { id: "T11", name: "A Thái", inRound: false },
                    { id: "D12", name: "Duy", inRound: false },
                    { id: "T13", name: "A Tư", inRound: false },
                    { id: "T14", name: "A Trường", inRound: false },
                    { id: "T15", name: "A Triết", inRound: false }
                ]
            }   
        },

        computed: {
            sortedMembers() {
                return [...this.members].sort((a, b) => b.coins - a.coins);
            }
        },

        created() {
            const savedMembers = localStorage.getItem('catte-members');
            const savedLogs    = localStorage.getItem('catte-logs');
            const savedNames   = localStorage.getItem('catte-names');

            if (savedMembers) this.members = JSON.parse(savedMembers);
            if (savedLogs)    this.logs    = JSON.parse(savedLogs);
            if (savedNames) {
                const parsed = JSON.parse(savedNames);
                // Merge saved inRound state back into the default names list
                this.names = this.names.map(n => {
                    const saved = parsed.find(s => s.id === n.id);
                    return saved ? { ...n, inRound: saved.inRound } : n;
                });
            }
        },

        watch: {
            members: {
                deep: true,
                handler(val) {
                    localStorage.setItem('catte-members', JSON.stringify(val));
                }
            },
            logs: {
                deep: true,
                handler(val) {
                    localStorage.setItem('catte-logs', JSON.stringify(val));
                }
            },
            names: {
                deep: true,
                handler(val) {
                    localStorage.setItem('catte-names', JSON.stringify(val));
                }
            }
        },

        methods: {
            addMember(){
                if (!this.member.name.trim()) {
                    this.$message({
                        type: 'error',
                        message: 'Name cannot be empty or whitespace.',
                    });
                    return;  
                }

                const newMember = Object.assign({}, this.member);
                newMember.id = this.generateIdFromCurrentDateTime();
                this.members.push(newMember);
                this.member.name = '';
            },

            addToRound(id, name) {
                const exists = this.members.some(member => member.nameId === id);
                if (exists) {
                    this.$message({
                        type: 'warning',
                        message: `${name} is already in the round.`,
                        });
                    return;  
                }

                const newMember = { ...this.member };  
                newMember.id = this.generateIdFromCurrentDateTime();  
                newMember.name = name;  
                newMember.nameId = id;

                this.members.push(newMember);

                const member = this.names.find(item => item.id === id);  
                if (member) {  
                    member.inRound = true;  
                } else {
                    this.$message({
                        type: 'error',
                        message: `Member with id ${id} not found in names list.`,
                    });
                }

                this.$message({
                    type: 'success',
                    message: `${name} has been added to the round.`,
                });
            },

            generateIdFromCurrentDateTime() {
                const currentDate = new Date();
                const id = currentDate.getFullYear() + 
                            (currentDate.getMonth() + 1).toString().padStart(2, '0') + 
                            currentDate.getDate().toString().padStart(2, '0') + 
                            currentDate.getHours().toString().padStart(2, '0') + 
                            currentDate.getMinutes().toString().padStart(2, '0') + 
                            currentDate.getSeconds().toString().padStart(2, '0') + 
                            currentDate.getMilliseconds().toString().padStart(3, '0');
                return id;
            },

            removeFromRound(id, name){
                this.$confirm(`Are you sure you want to remove ${name} from the round?`, 'Confirmation', {
                    confirmButtonText: 'Yes',
                    cancelButtonText: 'No',
                    type: 'warning',
                })
                    .then(() => {
                    this.members = this.members.filter(item => item.nameId !== id);
                    const memberName = this.names.find(name => name.id === id);
                    memberName.inRound = false;
                    
                    this.$message({
                            type: 'success',
                            message: `${name} has been removed from the round.`,
                        });
                    })
                    .catch(() => {
                        this.$message({
                            type: 'info',
                            message: 'Action canceled.',
                        });
                });
            },

            resetRound() {
                this.$confirm('Are you sure you want to reset the round? All members will be removed and coins will be cleared.', 'Reset Round', {
                    confirmButtonText: 'Yes, Reset',
                    cancelButtonText: 'Cancel',
                    type: 'warning',
                })
                    .then(() => {
                        this.members = [];
                        this.logs = [];
                        this.names.forEach(n => n.inRound = false);
                        this.$message({
                            type: 'success',
                            message: 'Round has been reset successfully.',
                        });
                    })
                    .catch(() => {
                        this.$message({
                            type: 'info',
                            message: 'Action canceled.',
                        });
                    });
            },

            addCoins(id, coins) {
                const member = this.members.find(member => member.id === id);
                if (!member) {
                    this.$message({
                        type: 'error',
                        message: 'Member not found!'
                    });
                    return;
                }

                this.$confirm(
                    `Are you sure you want to add ${coins} coins to ${member.name}?`,
                    'Confirmation',
                    {
                        confirmButtonText: 'Yes',
                        cancelButtonText: 'No',
                        type: 'warning'
                    }
                )
                    .then(() => {
                            member.coins += coins;

                            let log = {
                                index: this.logs.length + 1,
                                name: member.name,
                                coins: coins,
                                created: new Date().toLocaleString()
                            }

                            this.logs.push(log);

                            this.$message({
                                type: 'success',
                                message: `Added ${coins} coins to ${member.name} successfully!`
                            });
                        })
                    .catch(() => {
                        this.$message({
                            type: 'info',
                            message: 'Action canceled.'
                        });
                    });
            }
        }
    }
</script>