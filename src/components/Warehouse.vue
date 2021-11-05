<template>
  <h1>{{ msg }}</h1>

  <!--  Articles-->
  <div>
    <h2>Articles</h2>
    <a-button type="primary" @click="showAddArticleModal">
      Add article
    </a-button>
    <a-modal
        title="Add article"
        :visible="addArticleModalVisible"
        :confirm-loading="addArticleConfirmLoading"
        @ok="handleAddArticleOk"
        @cancel="handleAddArticleCancel"
    >
      <a-form :model="addArticleForm">
        <a-form-item label="Article name">
          <a-input v-model:value="addArticleForm.name"/>
        </a-form-item>
      </a-form>
    </a-modal>
    <a-row type="flex" justify="space-around" align="middle">
      <a-col :span="12">
        <a-table :columns="articleColumns" :data-source="articleData" row-key="articleId">
          <template #articleId="{record}">
            {{record.articleId}}
            <span>
              <copy-outlined @click="copyArticleId(record.articleId)"/>
            </span>
          </template>
        </a-table>
      </a-col>
    </a-row>
  </div>

  <!--  Shipment-->
  <div>
    <h2>Shipments</h2>
    <a-button type="primary" @click="showAddShipmentModal">
      Add shipment
    </a-button>
    <a-modal
        title="Add shipment"
        :visible="addShipmentModalVisible"
        :confirm-loading="addShipmentConfirmLoading"
        @ok="handleAddShipmentOk"
        @cancel="handleAddShipmentCancel"
    >
      <a-form :model="addShipmentForm">
        <a-form-item label="Customer name">
          <a-input v-model:value="addShipmentForm.customerName"/>
        </a-form-item>
        <a-form-item label="Volume">
          <a-input type="number" v-model:value="addShipmentForm.volume"/>
        </a-form-item>
        <a-form-item label="Article Id">
          <a-input v-model:value="addShipmentForm.articleId"/>
        </a-form-item>
      </a-form>
    </a-modal>
    <a-row type="flex" justify="space-around" align="middle">
      <a-col :span="18">
        <a-table :columns="shipmentColumns" :data-source="shipmentsData" row-key="shipmentId">
          <template #action="{record}">
            <span v-if="record.state === 'Ready to unload'">
              <import-outlined @click="showUnloadShipmentModal(record.shipmentId)"/>
            </span>
            <span v-if="record.state === 'Containers loaded, ready to be transported'">
              <export-outlined @click="departureShipment(record.shipmentId)"/>
            </span>
          </template>
        </a-table>
      </a-col>
    </a-row>
    <a-modal
        title="Unload Shipment"
        :visible="unloadShipmentModalVisible"
        @ok="handleUnloadShipmentOk"
        @cancel="handleUnloadShipmentCancel"
    >
      Unload all ({{ shipmentData.volume }}) containers
    </a-modal>
  </div>

  <!--  Containers-->
  <div>
    <h2>Containers</h2>
    <a-row type="flex" justify="space-around" align="middle">
      <a-col :span="18">
        <a-table :columns="containerColumns" :data-source="containersData" row-key="containerId">
          <template #reserved="{record}">
            <span v-if="record.reserved === true">
              <check-outlined/>
            </span>
            <span v-if="record.reserved === false">
              <close-outlined/>
            </span>
          </template>
          <template #action="{record}">
            <a-button
                v-if="record.reserved === true && record.location && record.location.includes('In truck for shipment') === false"
                @click="loadContainer(record)">Load
            </a-button>
            <a-button v-if="record.location && record.location.includes('In truck for shipment') === false"
                      @click="showMoveContainerModal(record.containerId)">Move
            </a-button>
          </template>
        </a-table>
      </a-col>
    </a-row>
    <a-modal
        title="Move Container"
        :visible="moveContainerModalVisible"
        @ok="handleMoveContainerOk"
        @cancel="handleMoveContainerCancel"
    >
      <a-form-item label="Location">
        <a-input v-model:value="containerData.location"/>
      </a-form-item>
    </a-modal>
  </div>

  <!--  Customer requests-->
  <div>
    <h2>Customer Requests</h2>
    <a-button type="primary" @click="showAddCustomerRequestModal">
      Add Customer request
    </a-button>
    <a-modal
        title="Add Customer request"
        :visible="addCustomerRequestModalVisible"
        @ok="handleAddCustomerRequestOk"
        @cancel="handleAddCustomerRequestCancel"
    >
      <a-form :model="addCustomerRequestForm">
        <a-form-item label="Customer">
          <a-input v-model:value="addCustomerRequestForm.customer"/>
        </a-form-item>
        <a-form-item label="Article Id">
          <a-input v-model:value="addCustomerRequestForm.articleId"/>
        </a-form-item>
        <a-form-item label="Volume">
          <a-input v-model:value="addCustomerRequestForm.volume"/>
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>

<script>
import axios from 'axios';
import {v4 as uuidv4} from 'uuid'
import {
  CloseOutlined,
  CopyOutlined,
  CheckOutlined,
  ExportOutlined,
  ImportOutlined,
  SelectOutlined
} from '@ant-design/icons-vue';

const articleColumns = [
  {
    title: 'Id',
    dataIndex: 'articleId',
    key: 'articleId',
    slots: {
      title: 'articleId',
      customRender: 'articleId'
    },
  },
  {
    title: 'Name',
    dataIndex: 'name',
    key: 'name',
  }
];
const shipmentColumns = [
  {
    title: 'Id',
    dataIndex: 'shipmentId',
    key: 'shipmentId'
  },
  {
    title: 'Customer Name',
    dataIndex: 'customerName',
    key: 'customerName',
  },
  {
    title: 'Volume',
    dataIndex: 'volume',
    key: 'volume',
  },
  {
    title: 'Article',
    dataIndex: 'articleId',
    key: 'articleId',
  },
  {
    title: 'State',
    dataIndex: 'state',
    key: 'state',
  },
  {
    title: 'Action',
    dataIndex: '',
    key: '',
    slots: {
      title: 'action',
      customRender: 'action'
    },
  }
];
const containerColumns = [
  {
    title: 'Id',
    dataIndex: 'containerId',
    key: 'containerId'
  },
  {
    title: 'Article name',
    dataIndex: 'articleName',
    key: 'articleName',
  },
  {
    title: 'Expiration date',
    dataIndex: 'expirationDate',
    key: 'expirationDate',
  },
  {
    title: 'Reserved',
    dataIndex: 'reserved',
    key: 'reserved',
    slots: {
      title: 'reserved',
      customRender: 'reserved'
    },
  },
  {
    title: 'Location',
    dataIndex: 'location',
    key: 'location',
  },
  {
    title: 'Action',
    dataIndex: '',
    key: '',
    slots: {
      title: 'action',
      customRender: 'action'
    },
  }
];

export default {
  name: 'Warehouse',
  props: {
    msg: String
  },
  created() {
    this.getArticles();
    this.getShipments();
    this.getContainers();
  },
  data() {
    return {
      articleColumns,
      shipmentColumns,
      containerColumns,
      articleData: [],
      shipmentsData: [],
      containersData: [],
      addArticleModalVisible: false,
      addArticleConfirmLoading: false,
      addArticleForm: {
        id: uuidv4(),
        name: ''
      },
      addShipmentModalVisible: false,
      addShipmentConfirmLoading: false,
      addShipmentForm: {
        id: uuidv4(),
        customerName: '',
        volume: 0,
        articleId: ''
      },
      unloadShipmentModalVisible: false,
      shipmentData: {},
      moveContainerModalVisible: false,
      containerData: {},
      addCustomerRequestModalVisible: false,
      addCustomerRequestForm: {
        shipmentId: uuidv4(),
        customer: '',
        articleId: '',
        volume: 0
      }
    }
  },
  methods: {
    async getArticles() {
      let result = await axios.get('http://localhost:8080/articles');
      this.articleData = result.data;
    },
    async getShipments() {
      let result = await axios.get('http://localhost:8080/shipments');
      this.shipmentsData = result.data;
    },
    async getContainers() {
      let result = await axios.get('http://localhost:8080/containers');
      this.containersData = result.data
    },
    async getShipment(id) {
      let result = await axios.get(`http://localhost:8080/shipments/${id}`);
      this.shipmentData = result.data;
    },
    async getContainer(id) {
      let result = await axios.get(`http://localhost:8080/containers/${id}`)
      this.containerData = result.data;
    },
    async loadContainer(record) {
      await axios.post('http://localhost:8080/articles/load-container', {
        articleId: record.articleId,
        shipmentId: record.reservedFor,
        containerId: record.containerId,
      })
      location.reload();
    },
    async departureShipment(id) {
      await axios.post(`http://localhost:8080/shipments/${id}/departure`)
      location.reload();
    },


    showAddArticleModal() {
      this.addArticleModalVisible = true;
    },
    showAddShipmentModal() {
      this.addShipmentModalVisible = true;
    },
    async showUnloadShipmentModal(id) {
      this.unloadShipmentModalVisible = true;
      await this.getShipment(id);
    },
    async showMoveContainerModal(id) {
      this.moveContainerModalVisible = true;
      await this.getContainer(id);
    },
    showAddCustomerRequestModal() {
      this.addCustomerRequestModalVisible = true;
    },
    copyArticleId(id) {
      var dummy = document.createElement("textarea");
      document.body.appendChild(dummy);
      //Be careful if you use texarea. setAttribute('value', value), which works with "input" does not work with "textarea". – Eduard
      dummy.value = id;
      dummy.select();
      document.execCommand("copy");
      document.body.removeChild(dummy);
    },

    async handleAddArticleOk(e) {
      e.preventDefault();
      this.addArticleConfirmLoading = true;
      await axios.post('http://localhost:8080/articles/create', {
        id: this.addArticleForm.id,
        name: this.addArticleForm.name
      })
      this.addArticleModalVisible = false;
      this.addArticleConfirmLoading = false;
      location.reload();
    },
    async handleAddShipmentOk(e) {
      e.preventDefault();
      this.addShipmentConfirmLoading = true;
      await axios.post('http://localhost:8080/shipments/create', {
        shipmentId: this.addShipmentForm.id,
        customerName: this.addShipmentForm.customerName,
        volume: this.addShipmentForm.volume,
        articleId: this.addShipmentForm.articleId,
      })
      this.addShipmentModalVisible = false;
      this.addShipmentConfirmLoading = false;
      location.reload();
    },
    async handleUnloadShipmentOk(e) {
      e.preventDefault();
      for (let i = 0; i < this.shipmentData.volume; i++) {
        await axios.post('http://localhost:8080/articles/unload-container', {
          articleId: this.shipmentData.articleId,
          shipmentId: this.shipmentData.shipmentId,
          containerId: uuidv4(),
          location: 'On Unloading dock',
          expirationDate: '2021-12-31T13:37:00.000Z'
        })
      }
      this.unloadShipmentModalVisible = false;
      location.reload();
    },
    async handleMoveContainerOk(e) {
      e.preventDefault();
      this.moveContainerModalVisible = false;
      await axios.post('http://localhost:8080/articles/move-container', {
        articleId: this.containerData.articleId,
        containerId: this.containerData.containerId,
        location: this.containerData.location
      })
      location.reload();
    },
    async handleAddCustomerRequestOk(e) {
      e.preventDefault();
      await axios.post('http://localhost:8080/shipments/initialise', {
        shipmentId: this.addCustomerRequestForm.shipmentId,
        articleId: this.addCustomerRequestForm.articleId,
        customer: this.addCustomerRequestForm.customer,
        volume: this.addCustomerRequestForm.volume
      })
      this.addCustomerRequestModalVisible = false;
      location.reload();
    },

    handleAddArticleCancel(e) {
      this.addArticleModalVisible = false;
    },
    handleAddShipmentCancel(e) {
      this.addShipmentModalVisible = false;
    },
    handleUnloadShipmentCancel(e) {
      this.unloadShipmentModalVisible = false;
    },
    handleMoveContainerCancel(e) {
      this.moveContainerModalVisible = false;
    },
    handleAddCustomerRequestCancel(e) {
      this.addCustomerRequestModalVisible = false;
    },
  },
  components: {
    CloseOutlined,
    CopyOutlined,
    CheckOutlined,
    ExportOutlined,
    ImportOutlined,
    SelectOutlined
  }
}
</script>
