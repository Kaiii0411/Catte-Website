<template>
    <div style="margin-bottom: 10px;">
    <el-form :model="member" label-width="auto" style="max-width: 600px">
        <el-form-item label="Name">
            <el-input v-model="member.name" />
        </el-form-item>
        <el-form-item>
            <el-button type="primary" @click="addMember()">Add</el-button>
        </el-form-item>
    </el-form>
    </div>
    <div class="card-container">
        <el-space wrap class="card-space" justify="center">
            <el-card v-for="member in members" :key="member" class="box-card" style="width: 180px;">
                <template #header>
                <div class="card-header">
                    <span><b>{{member.name}}</b></span>
                </div>
                </template>
                <div class="text item">
                    Coins: {{ member.coins }}
                </div>
                <template #footer>
                    <el-button class="button" type="primary" @click="addCoins(member.id, 5)">+ 5</el-button>
                    <el-button class="button" type="primary" @click="addCoins(member.id, 10)">+ 10</el-button>
                </template>
            </el-card>
        </el-space>
    </div>
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
                    coins: 0
                }
            }   
        },

        created() {

        },

        methods: {
            addMember(){
                const newMember = Object.assign({}, this.member);
                newMember.id = this.members.length + 1;
                this.members.push(newMember);
                this.member.name = '';
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