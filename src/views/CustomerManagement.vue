<template>
        <header class="customerM_header_title">
           <RouterLink :to="`/main`"><img src="@/assets/icon_arrow_left.svg" alt=""></RouterLink>
           <h3> 고객 관리</h3>
       </header>


       <section class="customerM_area">
            <div class="search">
                <input type="number" v-model="searchNum" name="" id="" placeholder="전화번호 뒤 4자리를 입력하세요.">
                
                    <button class="searchBtn" @click="search(searchNum)">검색</button>
                
            </div>

            <div class="select">
                <select id="member" name="member" v-model="selectedType" @change="updateFilteredCustomers">
                    <option selected="">전체</option>
                    <option value="일반">일반 고객</option>
                    <option value="단골">단골 고객</option>
                    <option value="추천">추천 고객</option>
                </select>
            </div>

            <div class="btn-group" role="group" aria-label="Basic example" style="margin-bottom:5px;">
                <button type="button" class="btn btn-outline-dark" id="customer_insert" @click="lover()" >단골 등록</button>
                <button type="button" class="btn btn-outline-dark" id="customer_insert2" @click="normal()" >일반 등록</button>
                <button type="button" class="btn btn-outline-success" id="coupon_gifts" @click="goPresentCoupon()">쿠폰 선물</button>
            </div>


            <ul class="list-group">
            <li class="list-group-item" style="border: 1px solid #000; background: linear-gradient(to right, #2BB488, #1A5CB7); color:white;">
                <div class="row">
                    <div class="col-2">
                        <input class="form-check-input me-1" type="checkbox" v-model="selectAll" @change="toggleSelectAll">
                    </div>
                    <div class="col-3">이름</div>
                    <div class="col-2">ID</div>
                    <div class="col-3">전화번호</div>
                    <div class="col-2">구분</div>
                </div>
            </li>
            
            <li class="list-group-item" style="border: 1px solid #000;" v-for="(customer, index) in filteredCustomers.length ? filteredCustomers : customers" :key="index">
                <div class="row">
                <div class="col-2">
                <input class="form-check-input me-1" type="checkbox" v-model="customer.checked">
                </div>
                <div class="col-3">{{ customer.name }}</div>
                <div class="col-2" style="transform: translateX(-15px);">{{ customer.id }}</div>
                <div class="col-3"  style="transform: translateX(-10px);" >{{ customer.phone }}</div>
                <div class="col-2">{{ customer.type }}</div>
                </div>
                </li>
                    </ul>
        </section>
</template>
<script>
import router from '@/router';
import { useResponseStore } from '@/store/response.js'


export default{
    name:'CustomerManagement',
    data(){
        return{
            customers:[],
            selectAll:false,
            searchNum:'',//검색통
            selectedType: '전체', // 고객 타입
            selectedCustomer:[]
        }
    },
    computed:{
        filteredCustomers() {
        return this.customers.filter(customer => {
            const matchesPhone = this.searchNum === '' || customer.phone.endsWith(this.searchNum);
            const matchesType = this.selectedType === '전체' || customer.type === this.selectedType;
       
            // 둘 다 값이 할당된 경우, 둘 다 만족해야 함
            if (this.searchNum !== '' && this.selectedType !== '전체') {
                return matchesPhone && matchesType;
            }

            // 하나라도 만족하는 경우
            return matchesPhone || matchesType; 
        });
    },
    watch: {
    filteredCustomers(newValue) {
        console.log('검색 결과:', newValue);
        console.log('선택된 타입:', this.selectedType);
    }
    }
},
    methods:{
        search(){
           
            console.log('전화번호 뒷자리 입력!!!!');

            this.updateFilteredCustomers();

        },
        toggleSelectAll() {
      this.customers.forEach(customer => {
        customer.checked = this.selectAll;
        });
        },
    lover(){
        console.log('단골고객');
        
    },
    normal(){
        console.log('일반고객');
        
    },
  
    goPresentCoupon(){
        //체크된 고객이 있는지 획인
        const selectedCustomers = this.customers.filter(customer => customer.checked);


        if(selectedCustomers.length === 0){
            alert('고객을 선택해주세요!')
            return;
        }
        const selectedIndex = this.customers
        .filter(customer => customer.checked)
        .map(customer => customer.store_customer_user_index)    
     
        console.log(selectedIndex);
        
        router.push({
            path:'/couponGift',
            query:{
              indexs:selectedIndex.join(','),
              length:selectedIndex.length
            }
        })
    },
    updateFilteredCustomers() {
    // 필요한 경우 추가적인 로직을 여기서 처리할 수 있습니다.
    console.log('고객 타입이 변경되었습니다:', this.selectedType);
    this.filteredCustomers; 
  },
    },
    mounted(){
        let store = useResponseStore();

        const formData = new FormData();
        
        formData.append('type','select')
        formData.append('user_index',store.user_index)

        const url = process.env.VUE_APP_API_URL;
        
        fetch(url + 'api/customer/customer_setting.php', {
                method: 'POST',
                body: formData
                })
                .then(response => response.json())
                .then(result =>{
                    console.log(result.msg);
                   
                    console.log(result.msg[0].store_customer_user_index);
                    
                    this.customers = result.msg.map(customer => ({
                        store_customer_user_index: customer.store_customer_user_index,
                        name:customer.name,
                        id:customer.user_id,
                        phone:customer.user_phone,
                        type: customer.store_customer_status,
                        checked: false
        
                    }))
                   
                
                    
                })
                
    



    }
}
</script>
<style scoped>

@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@100..900&display=swap');


*{
   font-family: "Noto Sans KR", sans-serif;
}

*ul,li{
    list-style: none;
    padding: 0;
}

.customerM_header_title{
   display: flex;
   align-items: center;
   justify-content: center; /* 가운데 정렬 */
   position: fixed;
   top: 0;
   left: 50%;
   width: 100%;
   /* max-width: 768px; */
   transform: translateX(-50%);
   height: 60px;
   background-color: #fff;
   font-size: 18px;
   font-weight: 800;
   border-bottom: 1px solid var(--line);
   z-index: 100;
}

.customerM_header_title > a {
   position: absolute; /* 왼쪽 버튼을 절대 위치로 */
   left: 10px; /* 왼쪽으로부터의 거리 */
   top: 50%; /* 세로 가운데 정렬 */
   transform: translateY(-50%); /* 세로 가운데 정렬 보정 */
}

.customerM_header_title > h3 {
   margin: 0;
   text-align: center; /* 텍스트 가운데 정렬 */
   color: #1749C2;
   font-weight: 900;
}

.customerM_area{
    width: 95%;
    margin: 80px auto 0;

}

.search{
    display: flex;
    justify-content: space-between;
    
    border-radius: 5px;
    margin-bottom: 5px;
}

.search input{
    width: 85%;
    border: 0;
    border: 1px solid #B1B1B1;

}

.searchBtn{
    width: 30%;
    padding: 10px;
    border: 0;
    border-radius: 5px;
    background-color: rgb(159, 76, 226);
    color: white;
}

select{
    width: 100%;
    padding: 5px; /* 여백을 추가해서 조금 더 넓어 보이게 설정 */
    border: 1px solid #B1B1B1; /* 테두리 설정 */
    border-radius: 5px; /* 모서리를 둥글게 설정 */
    font-size: 16px; /* 텍스트 크기를 적당히 조정 */
}

.btn-group{
    margin-top: 5px;
    display: flex;

}

.btn-group button{
    width: 35%;
    padding: 5px;
    background-color: #1749C2;
    color: white;
    border: 1px solid #f8f9fa;
    border-radius: 5px;
}
.list-group-item .row{
    display: flex;
    justify-content: space-around;
}
</style>