<template>
    <div class="product-detail">
        <div class="product-info">
            <div class="title">
                {{item.name}}
            </div>
            <div class="product-info-main">
                <div class="product-image">
                    <img :src="item.image || avatar" alt="">
                </div>
                <div class="product-property">
                   <table class="table-detail">
                       <tr><td class="table-title">Mã sản phẩm</td><td>{{item.id}}</td></tr>
                       <tr><td class="table-title">Xuất xứ</td><td>{{item.origin}}</td></tr>
                       <tr><td class="table-title">Đơn vị tính</td><td>{{item.unit}}</td></tr>
                       <tr><td class="table-title">Tình trạng</td><td>{{item.amount>0? "Còn hàng":"Hết hàng"}}</td></tr>
                   </table>
                   <p style="font-size:24px; color:#dc3545; padding: 24px 12px">Đơn giá: {{item.price}}đ</p>
                   <button class="button" @click="addCart(item)">Thêm vào giỏ hàng</button>
                </div>
                <div class="store-comment">
                    <p>DZ Office CAM KẾT:</p>
                    <div class="store-title">
                        <i class="fas fa-check-circle"></i>
                        Cung cấp các sản phẩm văn phòng đa dạng với giá cả hợp lý
                    </div>
                    <div class="store-title">
                        <i class="fas fa-check-circle"></i>
                        Miễn phí giao hàng cho đơn hàng trên 500.000đ (áp dụng tại các quận nội thành TP. Hà Nội)
                    </div>
                    <div class="store-title">
                        <i class="fas fa-check-circle"></i>
                        Hỗ trợ đổi trả nếu có lỗi sản phẩm hoặc không vừa ý
                    </div>
                    <div class="store-title">
                        <i class="fas fa-check-circle"></i>
                        Tư vấn và hướng dẫn sử dụng sản phẩm chi tiết, tận tình
                    </div>
                    <div class="store-title">
                        <i class="fas fa-check-circle"></i>
                        Cam kết chất lượng sản phẩm đúng với mô tả
                    </div>
                </div>
            </div>
        </div>
        <div class="product-info">
            <div class="title">
                Mô tả
            </div>
            <div class="main-description" v-html="item.description">
            </div>
        </div>
        <ErrorPopup :title="mesage" @close="close" v-if="hasError"></ErrorPopup>
        <Loader v-if="hasLoader"></Loader>
        <ToastMesage v-if="hasToast" :mesage="mesage" @closeToast="close"></ToastMesage>
    </div>
</template>
<script>
import Loader from '@/components/Bases/Loader'
import ToastMesage from '@/components/Bases/ToastMesage'
import ErrorPopup from '@/components/Bases/BasePopup/ErrorPopup'
import defaultAvatar from "../../assets/image/img_default.jpg"
import api from '@/js/api'
export default {
    components:{
        Loader,ToastMesage,ErrorPopup
    },
    data() {
        return {
            item:{},
            avatar: defaultAvatar,
            hasError:false,
            hasLoader:false,
            hasToast:false,
            mesage:""
        }
    },
    created(){
        this.$axios.get(`${api.ProductApi}/${this.getId}`).then(res=>{
            this.item = res.data
        })
    },
    computed:{
        getId(){
            return this.$route.params.id;
        }
    },
    methods:{
        addCart(item){
            let customer = this.$store.state.customer;
            if(customer){
                this.$axios.get(`${api.CartApi}/${customer.id}`).then(res =>{
                    if( res.data.some(c => c.cus_id==customer.id && c.prod_id==item.id)){
                        this.mesage="Sản phẩm này đã có trong giỏ hàng";
                        this.hasError= true;
                        return
                    }
                    let cart={
                    cus_id: customer.id,
                    prod_id: item.id,
                    quantity: 1
                    }
                    this.hasLoader=true;
                    this.$axios.post(api.CartApi,cart).then(res=>{
                        if(res.status == 201){
                            this.mesage = "Thêm vào giỏ hàng thành công."
                            this.hasToast=true;
                            setTimeout(() => {
                                this.hasToast=false
                            }, 3000);
                            this.$axios.get(`${api.CartApi}/${customer.id}`).then(res =>{
                                this.$store.commit("CHANGE_CART",res.data.length)
                            })
                        }
                    }).finally(()=>this.hasLoader=false)
                })             
            }
            else{
                this.$router.replace({path:'/signin'})
            }
            
        },
        close(){
            this.hasError=false;
            this.hasToast=false;
        }
    },
    watch:{
        getId: function(value){
          this.$axios.get(`${api.ProductApi}/${value}`).then(res=>{
            this.item = res.data;
        })  
        }
    }

}
</script>
<style scoped>
.title{
    border-bottom: 2px solid #ccc;
    height: 50px;
    padding: 0 24px;
    font-size: 32px;
    display: flex;
    align-items: flex-end;
    font-weight: 600;
    color: #3DAA12;
}
.product-detail{
    width: 100%;
    display: flex;
    flex-direction: column;
    gap: 24px;
}

.product-info{
    width: 100%;
    background: #fff;
    border-radius: 8px;
    margin-bottom: 24px;
}
.product-info-main{
    display: flex;
    gap: 12px;
    padding: 12px 24px;
    justify-content: space-between;
}
.product-info-main>*{
    flex-basis: 30%;
}
.main-description{
    padding: 12px 24px;
}

.product-image img{
    width: 100%;
}
.table-detail{
    border-collapse: collapse;
}
.table-detail td{
    border: 1px solid #ccc;
    min-width: 100px;
    padding: 8px 12px
}
.table-title{
    font-weight: 600;
}
.store-title{
    margin: 12px 0;
}
</style>