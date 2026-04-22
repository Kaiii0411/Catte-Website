<template>
    <el-tabs type="border-card">
        <!-- Tab Members -->
        <el-tab-pane label="Members">
            <div class="card-container" style="margin-bottom: 10px;">
                <el-table :data="names" border style="max-width: 600px">
                    <el-table-column label="Name" prop="name" />
                    <el-table-column label="Status">
                        <template #default="scope">
                            <el-tag :type="scope.row.inRound ? 'success' : 'danger'">
                            {{ scope.row.inRound ? 'In' : 'Out' }}
                            </el-tag>
                        </template>
                    </el-table-column>
                    <el-table-column>
                        <template #default="scope">
                            <el-button size="small"
                                type="success"
                                @click="addToRound(scope.row.id, scope.row.name)">
                                +
                            </el-button>
                            <el-button
                                size="small"
                                type="danger"
                                @click="removeFromRound(scope.row.id, scope.row.name)">
                                -
                            </el-button>
                        </template>
                        </el-table-column>
                </el-table>
            </div>
        </el-tab-pane>
        <!-- Tab Main -->
        <el-tab-pane label="Main">
            <el-collapse>
                <el-collapse-item title="Add">
                    <div class="card-container" style="margin-bottom: 10px;">
                    <el-form :model="member" label-width="auto" style="max-width: 600px">
                        <el-form-item label="Name">
                            <el-input v-model="member.name" />
                        </el-form-item>
                        <el-form-item style="float: right;">
                            <el-button type="primary" @click="addMember()">Add</el-button>
                        </el-form-item>
                    </el-form>
                </div>
                </el-collapse-item>
            </el-collapse>
            <div class="card-container" style="margin-top: 20px;">
                <el-space wrap class="card-space" justify="center">
                    <el-card v-for="member in members" :key="member" class="box-card" style="width: 350px;">
                        <template #header>
                        <div class="card-header">
                            <span><b>{{member.name}}</b></span>
                        </div>
                        </template>
                        <div class="text item">
                            Coins: 
                            <span :style="{ color: '#FF8C00'}">
                                {{ member.coins }}
                            </span>
                        </div>
                        <template #footer>
                            <el-button class="button" type="primary" @click="addCoins(member.id, 5)">+ 5</el-button>
                            <el-button class="button" type="primary" @click="addCoins(member.id, 10)">+ 10</el-button>
                            <el-button class="button" type="danger" @click="addCoins(member.id, -5)">- 5</el-button>
                            <el-button class="button" type="danger" @click="addCoins(member.id, -10)">- 10</el-button>
                        </template>
                    </el-card>
                </el-space>
            </div>
        </el-tab-pane>
        <!-- Tab Logs -->
        <el-tab-pane label="Logs">
            <div class="card-container" style="margin-bottom: 10px;">
                <el-table :data="logs" border style="max-width: 600px">
                    <el-table-column label="#" prop="index" />
                    <el-table-column label="Name" prop="name" />
                    <el-table-column label="Coins">
                        <template #default="{ row }">
                            <span :style="{ color: row.coins < 0 ? 'red' : 'green' }">
                            {{ row.coins }}
                            </span>
                        </template>
                        </el-table-column>
                    <el-table-column label="Created" prop="created" />
                </el-table>
            </div>
        </el-tab-pane>
        <!-- Tab Total -->
        <el-tab-pane label="Total">
            <div class="card-container" style="margin-bottom: 10px;">
                <el-table :data="members" border style="max-width: 600px">
                    <el-table-column label="#" type="index" width="50" />
                    <el-table-column label="Name" prop="name" />
                    <el-table-column label="Coins" prop="coins">
                        <template #default="scope">
                            {{ scope.row.coins + 10 }}
                        </template>
                    </el-table-column>
                </el-table>
            </div>
        </el-tab-pane>
    </el-tabs>
</template>

<style>
    .card-container {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100%; 
    }

    .card-space {
        width: 100%;
        justify-content: center; 
    }
</style>

<script type="text/javascript">
    export default {
        name: 'LogCoins',
        data() {
            return {
                members: [],
                member: {
                    id: 0,
                    name: '',
                    coins: 0,
                    nameId: ''
                },
                logs: [],
                names: [
                    { id: "N1", name: "Nghĩa", inRound: false},
                    { id: "T2", name: "C Tiền 🌸", inRound: false },
                    { id: "H3", name: "C Hương 🌻", inRound: false },
                    { id: "L4", name: "C Linh 🌷", inRound: false },
                    { id: "H5", name: "A Hiếu", inRound: false },
                    { id: "K6", name: "A Kiệt", inRound: false },
                    { id: "T7", name: "A Tân", inRound: false },
                    { id: "V8", name: "A Vinh", inRound: false },
                    { id: "V9", name: "A Vương", inRound: false },
                    { id: "T10", name: "C Thy", inRound: false },
                    { id: "T11", name: "A Thái", inRound: false }
                ]
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

                // Hiển thị thông báo thành công
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

            addCoins(id, coins) {
                const member = this.members.find(member => member.id === id);
                if (!member) {
                    this.$message({
                        type: 'error',
                        message: 'Member not found!'
                    });
                    return;
                }

                // Hiển thị hộp thoại xác nhận
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
                            // Nếu người dùng xác nhận, thêm coins
                            member.coins += coins;

                            let log = {
                                index: this.logs.length + 1,
                                name: member.name,
                                coins: coins,
                                created: new Date().toLocaleString()
                            }

                            this.logs.push(log);

                            // Hiển thị thông báo thành công
                            this.$message({
                                type: 'success',
                                message: `Added ${coins} coins to ${member.name} successfully!`
                            });
                        })
                    .catch(() => {
                        // Nếu người dùng hủy bỏ, thông báo đã hủy
                        this.$message({
                            type: 'info',
                            message: 'Action canceled.'
                        });
                    });
            }
        }
    }
</script>