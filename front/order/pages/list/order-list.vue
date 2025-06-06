<template lang="">
    <div class="background-image">
      <v-container>
        <!-- 주문 목록 있을 때 -->
        <v-row justify="center" v-if="orderStore.orderList.length > 0">
          <v-col cols="12" md="10" lg="8">
            <h2 class="title">
              <!-- <v-icon class="ml-auto" color="primary">mdi-clipboard-list-outline</v-icon> -->
              <span>🗒️ 구매한 보고서 목록</span>              
            </h2>
            <br>
            <v-data-table
              v-model:items-per-page="perPage"
              :headers="headerTitle"
              :items="pagedItems"
              v-model:pagination="pagination"
              class="elevation-1 order-table"
              @click:row="readRow"
              item-value="ordersId"
            />
  
            <!-- 페이지네이션 -->
            <v-row justify="center">
              <v-col cols="auto">
                <v-pagination
                  v-model="pagination.page"
                  :length="Math.ceil(orderStore.orderList.length / perPage)"
                  total-visible="7"
                  size="small"
                  color="primary"
                  @input="updateItems"
                />
              </v-col>
            </v-row>
          </v-col>
        </v-row>
  
        <!-- 주문 목록 없을 때 -->
        <v-row justify="center" v-else class="no-orders">
          <v-col cols="12" md="6">
            <v-card outlined class="pa-4 text-center">
              <v-icon size="64" color="grey lighten-1"
                >mdi-package-variant-closed</v-icon
              >
              <p class="no-orders-text">구매 기록이 없습니다.</p>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { useOrderStore } from '@/stores/orderStore'; // Pinia store import


// ✅ SEO 메타 정보
definePageMeta({
  title: '구매 목록 | JobStick',
  description: '사용자의 구매 목록을 확인할 수 있는 페이지입니다.',
  keywords: ['구매 목록', '구매', 'JobStick', '잡스틱', '구매 리스트', 'job-stick', '잡스틱'],
  ogTitle: 'JobStick 구매 목록',
  ogDescription: 'JobStick의 구매 목록 페이지입니다. 구매 기록을 확인하세요.',
  ogImage: '', // 실제 이미지 경로
  robots: 'index, follow' // 검색 노출 허용
});

const router = useRouter();
const orderStore = useOrderStore();

const headerTitle = [
  {
    title: "No",
    align: "start",
    sortable: true,
    key: "ordersId",
  },
  { title: "수량", align: "start", key: "totalQuantity", width: "20%" },
  { title: "금액", align: "start", key: "totalPrice", width: "40%" },
  {
    title: "구매 일자",
    align: "start",
    key: "createdDate",
    width: "40%",
  },
];

const perPage = ref(5);
const pagination = ref({ page: 1 });

const pagedItems = computed(() => {
  if (perPage.value === -1) {
    // Return the full list if items-per-page is set to 'all' (-1)
    return orderStore.orderList.map((item) => ({
      ...item,
      totalPrice: formatPrice(item.totalPrice), // Format totalPrice
      createdDate: formatDate(item.createdDate), // Format createdDate
    }));
  } else {
    const startIdx = (pagination.value.page - 1) * perPage.value;
    const endIdx = startIdx + perPage.value;
    return orderStore.orderList.slice(startIdx, endIdx).map((item) => ({
      ...item,
      totalPrice: formatPrice(item.totalPrice), // Format totalPrice
      createdDate: formatDate(item.createdDate), // Format createdDate
    }));
  }
});

function readRow(event, { item }) {
  router.push({
    name: "OrderReadPage",
    params: { ordersId: item["ordersId"].toString() },
  });
}

function formatDate(dateString) {
  const options = { year: "numeric", month: "2-digit", day: "2-digit" };
  const date = new Date(dateString);
  return date.toLocaleDateString("ko-KR", options);
}

function formatPrice(price) {
  return Math.round(price).toLocaleString(); // 반올림하고 천 단위로 콤마 추가
}

function goToLastPage() {
  router.go(-1);
}

onMounted(() => {
  orderStore.requestMyOrderListToDjango(); // Pinia store에서 직접 호출
});
</script>

<style scoped>
.background-image {
  margin-top: 7%;
}

.ml-auto {
  font-size: 100px !important;
}

.table-header th {
  font-size: 1.3em;
  font-weight: 900;
  color: #3f51b5;
}

.order-table {
  border-top: 1px solid #ddd;
  border-bottom: 1px solid #ddd;
}

.companyReport-image {
  max-width: 50px;
  max-height: 50px;
  object-fit: contain;
}

.no-orders {
  text-align: center;
  margin-top: 20px;
  font-size: 1.2em;
}

.no-orders-text {
  margin-top: 10px;
  color: #616161;
  font-size: 1.1em;
}
</style>
