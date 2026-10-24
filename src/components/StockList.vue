<template>
  <div class="container-fluid">
    <div class="row align-items-center justify-content-center">
      <div class="col col-12 align-items-center justify-content-center">
        <blockquote>
          Welcome {{ validUserName }}!
          <footer>
            <small>
              <em>&mdash; Financial Services, your Midwest Financial Services Partner.</em>
            </small>
          </footer>
        </blockquote>
      </div>
      <div class="col-12 col-md-10 col-lg-10 col-12 align-items-center justify-content-center">
        <div v-if="showMsg === 'new'" class="alert alert-success">
          New Stock has been added.
        </div>
        <div v-else-if="showMsg === 'update'" class="alert alert-success">
          Stock information has been updated.
        </div>
        <div v-else-if="showMsg === 'deleted'" class="alert alert-success">
          Selected stock has been deleted.
        </div>
      </div>
    </div>

    <!-- Mobile device view -->
    <div class="d-md-none" id="collapsable-card" style="width: 80%">
      <button
        v-if="authenticated"
        type="button"
        class="btn btn-primary"
        @click="addNewStock"
      >
        <i class="bi bi-plus"></i>
      </button>
      <button
        v-else
        type="button"
        class="btn btn-secondary"
        disabled
      >
        <i class="bi bi-plus"></i>
      </button>

      <div class="card" v-for="stock in stocks" :key="stock.pk">
        <div class="card-header" :id="'heading' + stock.pk">
          <button
            class="btn btn-link collapsed"
            data-bs-toggle="collapse"
            :data-bs-target="'#collapse' + stock.pk"
            aria-expanded="true"
            :aria-controls="'collapse' + stock.pk"
          >
            <h6 style="color: #0275d8; float: left">{{ stock.symbol }}</h6>
          </button>
        </div>
        <div
          :id="'collapse' + stock.pk"
          class="collapse"
          :aria-labelledby="'heading' + stock.pk"
          data-bs-parent="#collapsable-card"
        >
          <div class="card-body">
            <p><b>Customer:</b> {{ stock.customer }}</p>
            <p><b>Symbol:</b> {{ stock.symbol }}</p>
            <p><b>Name:</b> {{ stock.name }}</p>
            <p><b>Shares:</b> {{ stock.shares }}</p>
            <p><b>Purchase Price:</b> ${{ stock.purchase_price }}</p>
            <p><b>Purchase Date:</b> {{ stock.purchase_date }}</p>
            <div v-if="authenticated" class="btn-group">
              <button
                @click="updateStock(stock)"
                style="background-color: transparent; padding: 5px;"
              >
                <i class="bi bi-pencil"></i>
              </button>
              <button
                @click="deleteStock(stock)"
                style="background-color: transparent; padding: 5px;"
              >
                <i class="bi bi-trash"></i>
              </button>
            </div>
            <div v-else class="btn-group">
              <button style="background-color: transparent; padding: 5px;" disabled>
                <i class="bi bi-pencil"></i>
              </button>
              <button style="background-color: transparent; padding: 5px;" disabled>
                <i class="bi bi-trash"></i>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Non-mobile device view -->
    <div class="col col-12 col-md-10 d-none d-xl-block d-lg-block d-md-block">
      <table class="table table-hover" style="overflow-y: auto" :headers="headers">
        <thead>
          <tr>
            <th scope="col">Customer</th>
            <th scope="col">Symbol</th>
            <th scope="col">Name</th>
            <th scope="col">Shares</th>
            <th scope="col">Purchase Price</th>
            <th scope="col">Purchase Date</th>
            <th scope="col">Update</th>
            <th scope="col">Delete</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="stock in stocks" :key="stock.pk">
            <th scope="row">{{ stock.customer }}</th>
            <td>{{ stock.symbol }}</td>
            <td>{{ stock.name }}</td>
            <td>{{ stock.shares }}</td>
            <td>${{ stock.purchase_price }}</td>
            <td>{{ stock.purchase_date }}</td>
            <td v-if="authenticated" @click="updateStock(stock)">
              <button style="background-color: transparent; padding: 0;">
                <i class="bi bi-pencil"></i>
              </button>
            </td>
            <td v-else>
              <button style="background-color: transparent; padding: 0;" disabled>
                <i class="bi bi-pencil"></i>
              </button>
            </td>
            <td v-if="authenticated" @click="deleteStock(stock)">
              <button style="background-color: transparent; padding: 0;">
                <i class="bi bi-trash"></i>
              </button>
            </td>
            <td v-else>
              <button style="background-color: transparent; padding: 0;" disabled>
                <i class="bi bi-trash"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
      <button
        v-if="authenticated"
        type="button"
        class="btn btn-primary"
        @click="addNewStock"
      >
        Add New Stock
      </button>
      <button
        v-else
        type="button"
        class="btn btn-secondary"
        disabled
      >
        Add New Stock
      </button>
    </div>
  </div>
</template>

<script>
import router from '../router'
import { APIService } from '../api/APIService'
import { useAuthStore } from '@/store/AuthStore'

const apiService = new APIService()

export default {
  name: 'StockList',
  data: () => ({
    stocks: [],
    stockSize: 0,
    showMsg: '',
    isMobile: false,
    headers: [
      { text: 'Customer', sortable: false, align: 'left' },
      { text: 'Symbol', sortable: false, align: 'left' },
      { text: 'Name', sortable: false, align: 'left' },
      { text: 'Shares', sortable: false, align: 'left' },
      { text: 'Purchase Price', sortable: false, align: 'left' },
      { text: 'Purchase Date', sortable: false, align: 'left' },
      { text: 'Update', sortable: false, align: 'left' },
      { text: 'Delete', sortable: false, align: 'left' }
    ],
  }),
  computed: {
    authStore() {
      return useAuthStore()
    },
    authenticated() {
      return this.authStore.isAuthenticated
    },
    validUserName() {
      return this.authStore.user || 'Guest'
    }
  },
  mounted() {
    this.getStocks()
    this.showMessages()
  },
  methods: {
    onResize() {
      this.isMobile = window.innerWidth <= 600
    },
    showMessages() {
      if (this.$route.params.msg) {
        this.showMsg = this.$route.params.msg
      }
    },
    getStocks() {
      apiService
        .getStockList()
        .then((response) => {
          this.stocks = response.data.data
          this.stockSize = this.stocks.length
        })
        .catch((error) => {
          if (error.response && error.response.status === 401) {
            this.authStore.clearAuth()
            router.push('/auth')
          }
        })
    },
    addNewStock() {
      if (this.authenticated) {
        router.push('/stock-create')
      } else {
        router.push('/auth')
      }
    },
    updateStock(stock) {
      router.push('/stock-create/' + stock.pk)
    },
    deleteStock(stock) {
      if (confirm('Do you really want to delete?')) {
        apiService
          .deleteStock(stock.pk)
          .then((response) => {
            if (response.status === 204) {
              router.push('/stock-list/deleted/')
              this.getStocks()
            }
          })
          .catch((error) => {
            if (error.response && error.response.status === 401) {
              this.authStore.clearAuth()
              router.push('/auth')
            }
          })
      }
    }
  }
}
</script>

<style>
button {
  padding: 1rem;
  border: 0;
  cursor: pointer;
}
</style>>