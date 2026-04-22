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

            <div class="stake-row" style="justify-content: center; margin-bottom: 16px;">
              <span class="stake-label">Initial Stake per player</span>
              <el-input-number v-model="initialStake" :min="0" :step="1" size="small" style="width:130px" />
            </div>

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
            <div v-else class="total-wrap">

              <div class="table-wrap">
                <el-table :data="sortedMembers" class="catte-table total-table" border>
                  <el-table-column label="Rank" width="60" align="center">
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
                  <el-table-column label="Expected" width="105" align="center">
                    <template #default="scope">
                      <span style="font-weight:600;color:#94a3b8">{{ scope.row.finalBalance }}</span>
                    </template>
                  </el-table-column>
                  <el-table-column label="Actual" width="145" align="center">
                    <template #default="scope">
                      <el-input-number
                        :model-value="actualCounts[scope.row.id] !== undefined ? actualCounts[scope.row.id] : scope.row.finalBalance"
                        @update:model-value="val => actualCounts[scope.row.id] = val"
                        :step="1"
                        size="small"
                        style="width:120px"
                      />
                    </template>
                  </el-table-column>
                  <el-table-column label="Diff" width="85" align="center">
                    <template #default="scope">
                      <span :class="scope.row.diff > 0 ? 'coins-pos' : scope.row.diff < 0 ? 'coins-neg' : 'diff-zero'"
                            style="font-weight:700">
                        {{ scope.row.diff > 0 ? '+' + scope.row.diff : scope.row.diff === 0 ? '—' : scope.row.diff }}
                      </span>
                    </template>
                  </el-table-column>
                </el-table>
              </div>

              <div class="settle-area">
                <button class="settle-btn" @click="showSettlement = true">💰 Settle Up</button>
              </div>

            </div>
          </div>
        </el-tab-pane>

      </el-tabs>
    </main>

    <!-- ── Settlement Dialog ── -->
    <el-dialog v-model="showSettlement" title="💰 Settlement" width="520px">
      <div class="settlement-body">

        <!-- Surplus / Deficit summary -->
        <div v-if="surplusMembers.length > 0 || deficitMembers.length > 0" class="sd-summary">
          <div v-if="surplusMembers.length > 0" class="sd-group">
            <div class="sd-group-label surplus-label">📈 Surplus</div>
            <div v-for="m in surplusMembers" :key="m.id" class="sd-item">
              <span class="sd-name">{{ m.name }}</span>
              <span class="coins-pos sd-amount">+{{ m.diff }}</span>
            </div>
          </div>
          <div v-if="deficitMembers.length > 0" class="sd-group">
            <div class="sd-group-label deficit-label">📉 Deficit</div>
            <div v-for="m in deficitMembers" :key="m.id" class="sd-item">
              <span class="sd-name">{{ m.name }}</span>
              <span class="coins-neg sd-amount">{{ m.diff }}</span>
            </div>
          </div>
        </div>

        <div v-if="settlementTransactions.length > 0" class="sd-divider"></div>

        <!-- Case: no diffs at all -->
        <div v-if="surplusMembers.length === 0 && deficitMembers.length === 0" class="settlement-empty">
          <p>🎉 Everyone is balanced! No payments needed.</p>
        </div>

        <!-- Transfers -->
        <div v-if="settlementTransactions.length > 0">
          <div class="tx-section-label">Transfers</div>
          <div v-for="(tx, i) in settlementTransactions" :key="i" class="tx-row">
            <span class="tx-from">{{ tx.from }}</span>
            <span class="tx-arrow">gives</span>
            <span class="tx-to">{{ tx.to }}</span>
            <span class="tx-amount">{{ tx.amount }} 🪙</span>
          </div>
          <div class="settlement-check">
            <span :class="isBalanced ? 'check-ok' : 'check-warn'">
              {{ isBalanced ? '✅ All balanced — sum is zero' : '⚠️ Sum is not zero, check your data' }}
            </span>
          </div>
        </div>

      </div>
    </el-dialog>

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

/* ── Total Tab Layout ── */
.total-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.stake-row {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  max-width: 800px;
  justify-content: flex-end;
}

.stake-label {
  font-size: 0.85rem;
  font-weight: 600;
  color: #64748b;
  white-space: nowrap;
}

/* ── Settle Button ── */
.settle-area {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.settle-btn {
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  border: none;
  color: white;
  font-family: 'Outfit', sans-serif;
  font-size: 1rem;
  font-weight: 700;
  padding: 12px 36px;
  border-radius: 12px;
  cursor: pointer;
  letter-spacing: 0.3px;
  box-shadow: 0 4px 20px rgba(99, 102, 241, 0.35);
  transition: all 0.25s ease;
}

.settle-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 32px rgba(99, 102, 241, 0.55);
}

.settle-btn:active {
  transform: translateY(0);
}

/* ── Settlement Dialog ── */
.settlement-body {
  padding: 4px 0;
}

.settlement-empty {
  text-align: center;
  padding: 24px;
  color: #64748b;
  font-size: 1rem;
}

.tx-row {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 16px;
  margin-bottom: 8px;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.07);
  border-radius: 10px;
  flex-wrap: wrap;
}

.tx-from {
  font-weight: 700;
  color: #f87171;  /* giver = deficit = red */
  font-size: 0.95rem;
}

.tx-arrow {
  color: #64748b;
  font-size: 0.85rem;
  font-style: italic;
}

.tx-to {
  font-weight: 700;
  color: #4ade80;  /* receiver = surplus = green */
  font-size: 0.95rem;
}

.tx-amount {
  margin-left: auto;
  font-weight: 800;
  font-size: 1.05rem;
  color: #f0a500;
}

.settlement-check {
  margin-top: 16px;
  text-align: center;
  padding-top: 12px;
  border-top: 1px solid rgba(255,255,255,0.07);
  font-size: 0.9rem;
}

.check-ok   { color: #4ade80; font-weight: 600; }
.check-warn { color: #fbbf24; font-weight: 600; }

/* ── Total Table (wider) ── */
.total-table { max-width: 800px !important; }

/* ── Diff cell ── */
.diff-zero { color: #475569; font-weight: 600; }

/* ── Settlement surplus/deficit summary ── */
.sd-summary {
  display: flex;
  gap: 16px;
  margin-bottom: 4px;
  flex-wrap: wrap;
}

.sd-group {
  flex: 1;
  min-width: 180px;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.07);
  border-radius: 10px;
  padding: 12px 14px;
}

.sd-group-label {
  font-size: 0.78rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.6px;
  margin-bottom: 8px;
}

.surplus-label { color: #4ade80; }
.deficit-label { color: #f87171; }

.sd-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 4px 0;
  font-size: 0.9rem;
}

.sd-name { color: #e2e8f0; font-weight: 500; }
.sd-amount { font-weight: 700; font-size: 0.9rem; }

.sd-divider {
  height: 1px;
  background: rgba(255,255,255,0.07);
  margin: 16px 0;
}

.tx-section-label {
  font-size: 0.78rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.6px;
  color: #64748b;
  margin-bottom: 10px;
}
</style>

<script type="text/javascript">
    export default {
        name: 'LogCoins',
        data() {
            return {
                activeTab: 'members',
                initialStake: 10,
                showSettlement: false,
                actualCounts: {},
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
            // Enrich each member with finalBalance, actual entered, and diff
            memberDiffs() {
                return this.members.map(m => {
                    // coins now includes the initial stake directly
                    const finalBalance = m.coins;
                    const actual = (this.actualCounts[m.id] !== undefined)
                        ? this.actualCounts[m.id]
                        : finalBalance;
                    const diff = actual - finalBalance;
                    return { ...m, finalBalance, actual, diff };
                });
            },

            sortedMembers() {
                return [...this.memberDiffs].sort((a, b) => b.finalBalance - a.finalBalance);
            },

            surplusMembers() {
                return this.memberDiffs.filter(m => m.diff > 0);
            },

            deficitMembers() {
                return this.memberDiffs.filter(m => m.diff < 0);
            },

            // Deficit (diff < 0) means they owe points -> GIVERS
            // Surplus (diff > 0) means they earned points -> RECEIVERS
            settlementTransactions() {
                const givers    = this.deficitMembers.map(m => ({ name: m.name, balance: -m.diff })); // Make positive for math
                const receivers = this.surplusMembers.map(m => ({ name: m.name, balance: m.diff }));

                givers.sort((a, b) => b.balance - a.balance);
                receivers.sort((a, b) => b.balance - a.balance);

                const transactions = [];
                let i = 0, j = 0;

                while (i < givers.length && j < receivers.length) {
                    const giver    = givers[i];
                    const receiver = receivers[j];
                    const amount   = Math.min(giver.balance, receiver.balance);

                    transactions.push({ from: giver.name, to: receiver.name, amount });

                    giver.balance    -= amount;
                    receiver.balance -= amount;

                    if (Math.abs(giver.balance)    < 0.001) i++;
                    if (Math.abs(receiver.balance) < 0.001) j++;
                }

                return transactions;
            },

            isBalanced() {
                const total = this.memberDiffs.reduce((sum, m) => sum + m.diff, 0);
                return Math.abs(total) < 0.001;
            }
        },

        created() {
            const savedMembers = localStorage.getItem('catte-members');
            const savedLogs    = localStorage.getItem('catte-logs');
            const savedNames   = localStorage.getItem('catte-names');

            const savedStake   = localStorage.getItem('catte-stake');
            if (savedStake)   this.initialStake = JSON.parse(savedStake);
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
            },
            initialStake(val) {
                localStorage.setItem('catte-stake', JSON.stringify(val));
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
                newMember.initialStake = this.initialStake;
                newMember.coins = this.initialStake; // Start with the stake immediately

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
                        this.actualCounts = {};
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