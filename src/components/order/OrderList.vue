<template>
    <v-card
            class="mx-auto"
            outlined
    >
        <v-card-title>주문 내역</v-card-title>
        <v-data-table
                :loading="loadData"
                loading-text="Loading... Please wait"
                :headers="headers"
                :items="orderList"
                sort-by="orderId"
                class="elevation-1"
        >

            <template v-slot:item.action="{ item }">
                <v-chip :color="'green'"
                        dark
                        v-if="item.deliveryStarted && item.deliveryCompleted"
                > 배송 완료
                </v-chip>
                <v-chip :color="'green'"
                        dark
                        v-else-if="item.deliveryStarted && !item.deliveryCompleted"
                > 배송 시작
                </v-chip>
                <v-chip :color="'red'"
                        dark
                        v-else
                > 배송 준비 중
                </v-chip>
            </template>

            <template v-slot:item.Survey="{ item }">
                <v-icon
                        v-if="item.surveyCompleted"
                        small
                        @click="openSurvey(item)"
                >
                    리뷰보기
                </v-icon>

                <v-icon
                        v-else
                        small
                        @click="openSurvey(item)"
                >
                    리뷰작성
                </v-icon>
            </template>

        </v-data-table>
    </v-card>
</template>

<script>
    export default {
        name: 'OrderList',
        data: () => ({
            dialog: false,
            surveyComplete: true,
            loadData: false,
            headers: [
                {
                    text: '주문 번호',
                    align: 'center',
                    sortable: false,
                    value: 'orderId',
                },
                {
                    text: '주문상품',
                    align: 'center',
                    sortable: false,
                    value: 'productName',
                },
                {text: '구매수량', value: 'quantity', sortable: false, align: 'center'},
                {text: '결제금액', value: 'payment',sortable: false, align: 'center'},
                {text: 'Delivery', value: 'action', sortable: false, align: 'center'},
                {text: '리뷰', value: 'Survey', sortable: false, align: 'center'},
                // {text: '결제시각', value: 'timestamp', align: 'center'},
            ],
            orderList: [],
            surveyList:[],
        }),
        computed: {
        },
        watch: {
            dialog(val) {
                val || this.close()
            },
        },

        created() { },
        mounted() {
            this.mountedFunction()
        },
        methods: {
            getSurveyList(){
                var me = this
                if(`${API_HOST}` == 'undefined')
                    window.API_HOST = localStorage.getItem('api_host')

                return new Promise(function (resolve, reject) {
                    me.$http.get(`${API_HOST}/surveys`).then(function (e) {
                        resolve(e.data._embedded.surveys)
                    });
                });


            },
            getOrderList() {
                var me = this
                if(`${API_HOST}` == 'undefined')
                    window.API_HOST = localStorage.getItem('api_host')

                return new Promise(function (resolve, reject) {
                    me.$http.get(`${API_HOST}/mypage/order/${localStorage.getItem('userId')}`).then(function (e) {
                        resolve(e.data)
                    });
                });

            },
            async mountedFunction() {
                var me = this;

                var order = await me.getOrderList();
                var sur = await me.getSurveyList();

                order.forEach(function(or){
                    sur.forEach(function(select){
                        if(select.orderId == or.orderId){
                            or.surveyCompleted=true
                        }
                    })
                })

                me.orderList=order
            },
            openSurvey(item) {
                var me = this
                let check=false

                this.$http.get(`${API_HOST}/surveys`).then(function (surveyList) {

                    surveyList.data._embedded.surveys.some(function(select){
                            if(select.orderId == item.orderId){
                                check = true
                                return me.$router.push({ name: 'surveys', params: select });
                            }
                        })


                    if(check == false){
                        var data= {
                            'orderId' : item.orderId,
                            'customerId': item.userId,
                            'customerName': item.nickName,
                            'productName': item.productName,
                            'productPrice': item.payment,
                            'productQty': item.quantity,
                            'surveyMessage': '',
                            'surveyRecommend':2,
                            'surveyDelivery':2,
                            'productSatisfaction': 2,
                            'surveyCompleted': false
                        }
                        me.$router.push({ name: 'surveys', params: data });
                    }

                })



            },

        },
    }
</script>

<style scoped>

</style>
