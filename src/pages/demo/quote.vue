<template>
  <NuxtLayout name="empty">
    <section class="fixed inset-0 grid grid-cols-[4fr_3fr_3fr] bg-#f5f5f7 p-6 gap-6 justify-center">
      <header class="h-6 font-600 text-lg">后端接口返回 JSON -> </header>
      <header class="h-6 font-600 text-lg">JSON -> 黑盒计算逻辑 -> BillInfo</header>
      <header class="h-6 font-600 text-lg">Quote 账单</header>

      <section class="flex flex-col gap-6 overflow-y-auto">
        <Accordion multiple :value="['bosList', 'acList']">
          <AccordionPanel value="KeyProductList">
              <AccordionHeader>主商品: KeyProductList: Array</AccordionHeader>
              <AccordionContent>
                <Button class="w-14 h-6 text-xs" @click="syncData('keyProductList')">Sync</Button>
                <div contenteditable>
                  <pre data-area="keyProductList">{{ keyProductList }}</pre>
                </div>
              </AccordionContent>
          </AccordionPanel>
          <AccordionPanel value="bosList">
              <AccordionHeader>BoS bosList: Array</AccordionHeader>
              <AccordionContent>
                <Button class="w-14 h-6 text-xs" @click="syncData('bosList')">Sync</Button>
                <div contenteditable>
                  <pre data-area="bosList">{{ bosList }}</pre>
                </div>
              </AccordionContent>
          </AccordionPanel>
          <AccordionPanel value="acList">
              <AccordionHeader>AC acList: Array</AccordionHeader>
              <AccordionContent>
                <Button class="w-14 h-6 text-xs" @click="syncData('acList')">Sync</Button>
                <div contenteditable>
                  <pre data-area="bosList">{{ acList }}</pre>
                </div>
              </AccordionContent>
          </AccordionPanel>
          <AccordionPanel value="customDeductionList">
              <AccordionHeader>自定义补贴: customDeductionList: Array</AccordionHeader>
              <AccordionContent>
                <Button class="w-14 h-6 text-xs" @click="syncData('customDeductionList')">Sync</Button>
                <div contenteditable>
                  <pre data-area="customDeductionList">{{ customDeductionList }}</pre>
                </div>
              </AccordionContent>
          </AccordionPanel>
          <AccordionPanel value="fixedDeduction">
              <AccordionHeader>固定补贴 fixedDeduction: object</AccordionHeader>
              <AccordionContent>
                现业务只支持澳洲，按洲区分，可扩展，后续可支持多个国家
                <div contenteditable>
                  <pre>{{ defaultDeductions }}</pre>
                </div>
              </AccordionContent>
          </AccordionPanel>
        </Accordion>

        <Card>
          <template #header>
            <h2 class="p-6 pb-0">Caclulator 类及相关使用</h2>
          </template>
          <template #content>
            <p>
              CalcInst.decimalToPercent(0.50549993) ->
              {{ CalcInst.decimalToPercent(0.50559876) }}
            </p>
            <p>
              CalcInst.decimalToPercent(0.50549993, 4) -> 
              {{ CalcInst.decimalToPercent(0.50559876, 4) }}
            </p>
            <p>
              CalcInst.percentToDecimal(50.56) ->
              {{ CalcInst.percentToDecimal(50.56) }}
            </p>
            <p>
              CalcInst.percentToDecimal(50.567890, 4) ->
              {{ CalcInst.percentToDecimal(50.567890, 4) }}
            </p>
            <p>
              CalcInst.subtractMultiple(15, [1,2,3,4, 5.5]) ->
              {{ CalcInst.subtractMultiple(15, [1,2,3,4,5.5]) }}
            </p>

            <div>
              看一下缓存状态：
              <pre>{{ cacheData }}</pre>
            </div>
          </template>
        </Card>
      </section>

      <section class="flex flex-col gap-6 overflow-y-auto">
        <Accordion multiple :value="['finalGroup', 'billInfo']">
          <AccordionPanel value="billInfo">
              <AccordionHeader>账单小计 billInfo: Computed</AccordionHeader>
              <AccordionContent>
                <Button class="w-25 h-6 text-xs" :disabled="true" @click="syncData('billInfo')">Can't Sync</Button>
                <div :contenteditable="false">
                  <pre data-area="billInfo">{{ billInfo }}</pre>
                </div>
              </AccordionContent>
          </AccordionPanel>
          <AccordionPanel value="finalGroup">
              <AccordionHeader>用户输入 finalGroup: Ref</AccordionHeader>
              <AccordionContent>
                <Button class="w-25 h-6 text-xs" :disabled="true" @click="syncData('finalGroup')">Auto Sync</Button>
                <div :contenteditable="false">
                  <pre data-area="finalGroup">{{ finalGroup }}</pre>
                </div>
              </AccordionContent>
          </AccordionPanel>
        </Accordion>
      </section>

      <section class="flex flex-col overflow-y-auto gap-6">
        <Card class="bg-white p-0">
          <template #header>
            <h2 class="p-4 pb-0">单项数据流 > BillInfo 数据展示如下：</h2>
          </template>
          <template #content>
            <div class="item-row font-600 pr-6 cursor-pointer" @click="isTotalFold = !isTotalFold">
              <label>
                <span>{{ t('system_total_inclgst') }}</span>
                <small class="ml-1 text-grey-light text-xs font-400">{{ t('incl_tax_name', { taxName: 'GST' }) }}</small>
              </label>
              <span>{{ formatMoney(finalGroup.fixedPriceFlag ? billInfo.fixedSystemPrice : billInfo.systemTotal) }}</span>
              <Icon name="material-symbols:keyboard-arrow-down-rounded" size="1.2rem"
                class="text-grey-light absolute right-0 top-0 transition-transform duration-300"
                :class="isTotalFold ? 'rotate-0' : 'rotate-180'" />
            </div>
            <section v-show="!isTotalFold" class="use-transition">
              <div class="item-row pl-4 pr-6">
                <label class="text-xs">{{ t('q_key_products') }} <span class="text-grey-light ">{{ t('excl_tax_name', { taxName: 'GST' }) }}</span></label>
                <!-- <span class="text-xs">{{ isFixedPrice === OPEN ? '-' : formatDisplay.kpPrice }}</span> -->
                <span class="text-xs">{{ formatMoney(finalGroup.fixedPriceFlag ? billInfo.fixedKpPrice : billInfo.kpPrice) }}</span>
              </div>
              <div class="item-row pl-4 pr-6">
                <label class="text-xs">{{ t('q_balance_of_system') }} <span class="text-grey-light ">{{ t('excl_tax_name', { taxName: 'GST' }) }}</span></label>
                <span class="text-xs">{{ formatMoney(billInfo.bosPrice) }}</span>
              </div>
              <div class="item-row pl-4 pr-6">
                <label class="text-xs">{{ t('q_additional_charges') }} <span class="text-grey-light ">{{ t('excl_tax_name', { taxName: 'GST' }) }}</span></label>
                <span class="text-xs">{{ formatMoney(billInfo.acPrice) }}</span>
              </div>
              <div class="item-row pl-4 pr-6">
                <label class="text-xs">GST <span class="text-grey-light">({{ taxRate }})</span></label>
                <span class="text-xs">{{ formatMoney(finalGroup.fixedPriceFlag ? billInfo.fixedSystemRate : billInfo.systemTaxRate) }}</span>
              </div>
            </section>

            <div class="item-row font-600 pr-6 cursor-pointer" @click="isRebateFold = !isRebateFold">
              <label>
                <span>{{ t('t_deductions') }}</span>
              </label>
              <span class="text-green flex-1 text-right whitespace-nowrap">
                - {{ formatMoney(billInfo.customDeductionPriceInclRate) }}
              </span>
              <Icon name="material-symbols:keyboard-arrow-down-rounded" size="1.2rem"
                class="text-gray absolute right-0 top-0 use-transition"
                :class="isRebateFold ? 'rotate-0' : 'rotate-180'" />
            </div>

            <section v-show="!isRebateFold">
              <!-- <div v-show="auDeductionDetails?.stcVisible" class="item-row pl-4 pr-6">
                <label class="text-xs">
                  STC - Panel
                  <span class="text-gray">
                    ({{ formatDisplay.stcUnitPrice }}
                    × {{ auDeductionDetails?.stcQuantity }};
                    {{ auDeductionDetails?.stcGstType === 'gstFree' ? 'GST free' : auDeductionDetails?.stcGstType === 'exclGst' ? 'excl. GST' : 'incl. GST' }})
                  </span>
                </label>
                <span class="text-green text-xs flex-1 text-right whitespace-nowrap">{{ formatDisplay.stcTotalPrice }}</span>
              </div> -->

              <!-- <div v-show="auDeductionDetails?.supportStcBattery && auDeductionDetails?.stcBatterySwitch" class="item-row pl-4 pr-6">
                <label class="text-xs">
                  STC - Battery
                  <span class="text-gray">
                    ({{ formatDisplay.stcBatteryUnitPrice }}
                    × {{ auDeductionDetails?.stcBatteryQty }};
                    {{ auDeductionDetails?.stcBatteryGstType === 'gstFree' ? 'GST free' : auDeductionDetails?.stcBatteryGstType === 'exclGst' ? 'excl. GST' : 'incl. GST' }})
                  </span>
                </label>
                <span class="text-green text-xs flex-1 text-right whitespace-nowrap">{{ formatDisplay.stcBatteryTotalPrice }}</span>
              </div> -->

              <!-- <div v-show="auDeductionDetails?.supportVic && auDeductionDetails?.vicVisible" class="item-row pl-4 pr-6">
                <label class="text-xs">
                  Solar VIC Rebate
                  <span class="text-gray">({{ formatDisplay.vicGstType }})</span>
                </label>
                <span class="text-green text-xs flex-1 text-right whitespace-nowrap">{{ formatDisplay.vicTotal }}</span>
              </div> -->

              <!-- <div v-show="auDeductionDetails?.supportVicPv && auDeductionDetails?.vicPvVisible" class="item-row pl-4 pr-6">
                <label class="text-xs">
                  Solar VIC PV Interest Free Loan
                  <span class="text-gray">({{ formatDisplay.vicPvGstType }})</span>
                </label>
                <span class="text-green text-xs flex-1 text-right whitespace-nowrap">{{ formatDisplay.vicPvTotal }}</span>
              </div> -->

              <!-- <div v-show="auDeductionDetails?.supportVicBi && auDeductionDetails?.vicBiVisible" class="item-row pl-4 pr-6">
                <label class="text-xs">
                  Solar VIC Battery Interest Free Loan
                  <span class="text-gray">({{ formatDisplay.vicBiGstType }})</span>
                </label>
                <span class="text-green text-xs flex-1 text-right whitespace-nowrap">{{ formatDisplay.vicBiTotal }}</span>
              </div> -->

              <!-- <div v-show="auDeductionDetails?.supportBess && auDeductionDetails?.bess1Visible" class="item-row pl-4 pr-6">
                <label class="text-xs">
                  NSW Battery Rebate (BESS1)<br>
                  <span class="text-gray">(formatDisplay.bess1UnitPrice
                    × {{ auDeductionDetails?.bess1Quantity }};
                    {{ auDeductionDetails?.bess1GstType === 'exclGst' ? 'excl. GST' : 'incl. GST' }})</span>
                </label>
                <span class="text-green text-xs flex-1 text-right whitespace-nowrap">{{ formatDisplay.bess1TotalPrice }}</span>
              </div> -->

              <!-- <div v-show="auDeductionDetails?.supportBess2 && auDeductionDetails?.bess2Visible" class="item-row pl-4 pr-6">
                <label class="text-xs">
                  NSW VPP Rebate (BESS2)<br>
                  <span class="text-gray">({{ formatDisplay.bess2UnitPrice }}
                    × {{ auDeductionDetails?.bess2Quantity }};
                    {{ auDeductionDetails?.bess2GstType === 'exclGst' ? 'excl. GST' : 'incl. GST' }})</span>
                </label>
                <span class="text-green text-xs flex-1 text-right whitespace-nowrap">{{ formatDisplay.bess2TotalPrice }}</span>
              </div> -->
              <DetuctionDisplayItem v-for="item in customDeductionList" :key="item.uid"
                :data="item" :gst="taxRate"
                class="gap-0!" label-class="text-typo-main!"
                item-class="mb-2 rebate-item item-row__sub pr-6" />

              <div v-show="billInfo.customDeductionRate" class="item-row pl-4 pr-6">
                <label class="text-xs">GST <span class="text-gray">({{ taxRate }})</span></label>
                <span class="text-green text-xs flex-1 text-right whitespace-nowrap">
                  - {{ billInfo.customDeductionRate }}
                </span>
              </div>
            </section>
          </template>
        </Card>

        <Card class="flex flex-col gap-2 pb-4">
          <template #header>
            <p class="p-4 pb-0">
              1. 即时修改同步计算 -> 到4个输入框<br/>
              2. finalPrice 改变，重新进入黑盒逻辑 -> 计算账单<br/>
              3. 新账单，渲染结果如上，并同步用户输入 <br/>
            </p>
          </template>
          <template #content>
            <div class="user-input-area flex flex-col gap-y-2">
              <div class="flex flex-col gap-y-2">
                <div class="grid grid-cols-[3fr_2fr]">
                  <label>totalCost: </label>
                  <div class="text-right" v-show="!finalGroup.fixedPriceFlag">{{ formatMoney(billInfo.totalCost) }}</div>
                  <InputNumber v-show="finalGroup.fixedPriceFlag" v-model="finalGroup.fixedTotalCost" 
                    prefix="$ " locale="en-US"
                    :min-fraction-digits="2" :max-fraction-digits="2"
                    @update:model-value="setFinalGroupField('totalCost', $event)" />
                </div>
                <div v-show="finalGroup.fixedPriceFlag" class="auto-item">
                  {{ t('auto_cost') }}: {{ formatMoney(billInfo.totalCost) }}
                </div>
              </div>
              <div class="flex flex-col gap-y-2">
                <div class="grid grid-cols-[3fr_2fr]">
                  <label>grossMargin: </label>
                  <div class="text-right" v-show="!finalGroup.fixedPriceFlag">{{ formatMoney(billInfo.grossMargin) }}</div>
                  <InputNumber v-show="finalGroup.fixedPriceFlag" v-model="finalGroup.grossMargin" 
                    :min="1" :max="99"
                    :min-fraction-digits="2" :max-fraction-digits="2"
                    suffix="%" locale="en-US"
                    @update:model-value="setFinalGroupField('grossMargin', $event)" />
                </div>
                <div v-show="finalGroup.fixedPriceFlag" class="auto-item">
                  {{ t('auto_margin') }}: {{ billInfo.grossMargin ? billInfo.grossMargin + '%' : billInfo.grossMargin }}
                </div>
              </div>
              <div class="flex flex-col gap-y-2">
                <div class="grid grid-cols-[3fr_2fr]">
                  <label>grossProfit: </label>
                  <div class="text-right" v-show="!finalGroup.fixedPriceFlag">{{ formatMoney(billInfo.grossProfit) }}</div>
                  <InputNumber v-show="finalGroup.fixedPriceFlag" v-model="finalGroup.grossProfit"
                    prefix="$ " locale="en-US"
                    :min-fraction-digits="2" :max-fraction-digits="2"
                    @update:model-value="setFinalGroupField('grossProfit', $event)" />
                </div>
                <div v-show="finalGroup.fixedPriceFlag" class="auto-item">
                  {{ t('auto_profit') }}: {{ formatMoney(billInfo.grossProfit) }}
                </div>
              </div>
              <div class="flex flex-col gap-y-2">
                <div class="grid grid-cols-[3fr_2fr]">
                  <label>finalPrice: </label>
                  <InputNumber class="text-right" v-model="finalGroup.finalPrice" 
                    prefix="$ " locale="en-US"
                    :min-fraction-digits="2" :max-fraction-digits="2"
                    @update:model-value="changeFinalPrice" />
                </div>
                <div v-show="finalGroup.fixedPriceFlag" class="auto-item">
                  {{ t('auto_price') }}: {{ formatMoney(billInfo.finalPrice) }}
                </div>
                <div v-show="finalGroup.fixedPriceFlag" class="mt-1 text-right">
                  <span>isFixedPrice: {{  finalGroup.fixedPriceFlag ? '1' : '0' }}，当前为 {{ finalGroup.fixedPriceFlag ? '一口价' : '非一口价' }}</span>
                  <span class="ml-2 text-blue cursor-pointer ml-1" @click="handleReset">
                    {{ t('reset_button') + ' ' + t('all') }}
                  </span>
                </div>
              </div>
            </div>
          </template>
        </Card>

        <Card>
          <template #content>
            <ComputedInfo class="mt-2 rounded bg-white"
              project-type="system_design"
              :loading="isFetching"
              :card-info="calculatedKeyInfo"
            />
          </template>
        </Card>
      </section>
    </section>
  </NuxtLayout>
</template>

<script setup lang="ts">
import useQuoteFetch from '~/composables/projects/quote/useQuoteFetch'
import ComputedInfo from '../projects/[projectId]/@components/quote/ComputedInfo.vue'
import DetuctionDisplayItem from '../projects/[projectId]/@components/quote/DetuctionDisplayItem.vue'
import usePriceCalcator from '~/composables/projects/quote/usePriceCalcator'

const { t } = useI18n()

function handleReset() {
  console.log('🚀 ~ handleReset:')
  finalGroup.value.fixedPriceFlag = 0
  initData()
}

const isTotalFold = ref(false)
const isRebateFold = ref(false)

const { 
  taxRate,
  keyProductList, setKeyProductList,
  bosList, setBosList,
  acList, setAcList,
  customDeductionList, setCustomDeductionList,
  defaultDeductions, setDefaultDeductions,
  finalGroup, setFinalGroupField, billInfo,
} = usePriceCalcator()
const { 
  calculatedKeyInfo, // setCalculatedKey,
} = useQuoteFetch()

type JsonKey = 
  | 'finalGroup'
  | 'billInfo'
  | 'keyProductList'
  | 'bosList'
  | 'acList'
  | 'customDeductionList'
function syncData(jsonKey: JsonKey) {
  const element = document.querySelector(`[data-area=${jsonKey}]`)
  const jsonStrToObject = JSON.parse(element?.textContent as string)
  console.log('🚀 ~ syncData:', jsonStrToObject)
  
  switch (jsonKey) {
    case 'keyProductList':
      setKeyProductList(jsonStrToObject)
      break
    case 'acList':
      setAcList(jsonStrToObject)
      break
    case 'bosList':
      setBosList(jsonStrToObject)
      break
    case 'customDeductionList':
      setCustomDeductionList(jsonStrToObject)
      break
    default:
      break
  }
  element?.textContent
}

function changeFinalPrice(evt) {
  console.log('🚀 ~ changeFinalPrice:', evt)
  finalGroup.value.fixedPriceFlag = 1
  setFinalGroupField('finalPrice', evt)
}

function initData() {
  setKeyProductList([
      {
          "id": 2373,
          "projectId": 1414,
          "projectDesignId": 1251,
          "cartType": "design",
          "productId": 1568,
          "productSource": 1,
          "productType": "mounting",
          "productName": "ALU-MOUNT-RAIL-L5900-EU",
          "productInfo": "{\"brandLogo\":{\"fileName\":\"logo_36d81e45222e4ec366cd1e07cdf52ea3.png\",\"originalFilePath\":\"https://assets.solarbrain.com.au/uploads/photo_attachment/photo/1847/logo_36d81e45222e4ec366cd1e07cdf52ea3.png\",\"thumbnailFilePath\":\"https://assets.solarbrain.com.au/uploads/photo_attachment/photo/1847/logo_36d81e45222e4ec366cd1e07cdf52ea3.png\",\"contentType\":1},\"brandName\":\"Alufix\",\"brandId\":325,\"productName\":\"ALU-MOUNT-RAIL-L5900-EU\",\"productCode\":\"ALU-MOUNT-RAIL-L5900-EU\",\"productPhotos\":[{\"fileName\":\"image-not-found.png\",\"originalFilePath\":\"https://file.greensketch.ai/static/images/image-not-found.png\",\"thumbnailFilePath\":\"https://file.greensketch.ai/static/images/image-not-found.png\",\"contentType\":1}]}",
          "quantity": 1,
          "unitCost": 33.33,
          "unitPrice": 400,
          "margin": 91.67,
          "lineCost": 33.33,
          "linePrice": 400,
          "materialsChangedStatus": 0,
          "proposalVisibleValues": null
      },
      {
          "id": 2374,
          "projectId": 1414,
          "projectDesignId": 1251,
          "cartType": "design",
          "productId": 225,
          "productSource": 2,
          "productType": "panel",
          "productName": "AE655ME-132",
          "productInfo": "{\"brandLogo\":{\"fileName\":\"AE+Solar.png\",\"originalFilePath\":\"https://file.greensketch.net/brands/logo/AE+Solar.png\",\"thumbnailFilePath\":\"https://file.greensketch.net/brands/logo/AE+Solar.png\",\"contentType\":1},\"brandName\":\"AE Solar\",\"brandId\":246,\"productName\":\"AE655ME-132\",\"productCode\":\"AE655ME-132\",\"oswItemCodeAu\":\"/\",\"oswItemCodeEu\":\"/\",\"productPhotos\":[{\"fileName\":\"AE_Solar_AE655ME-132.jpg\",\"originalFilePath\":\"https://file.greensketch.net/images/panels/AE_Solar_AE655ME-132.jpg\",\"thumbnailFilePath\":\"https://file.greensketch.net/images/panels/AE_Solar_AE655ME-132.jpg\",\"contentType\":1}],\"productWarranty\":\"15\",\"performanceWarranty\":\"30\",\"dataSheets\":[{\"fileName\":\"AE_Solar_AE655ME-132.pdf\",\"originalFilePath\":\"https://file.greensketch.net/datasheets/panels/AE_Solar_AE655ME-132.pdf\",\"thumbnailFilePath\":\"https://file.greensketch.net/datasheets/panels/AE_Solar_AE655ME-132.pdf\",\"contentType\":1}],\"warrantyDocuments\":[{\"fileName\":\"AE_Solar_AE655ME-132.pdf\",\"originalFilePath\":\"https://file.greensketch.net/warranties/panels/AE_Solar_AE655ME-132.pdf\",\"thumbnailFilePath\":\"https://file.greensketch.net/warranties/panels/AE_Solar_AE655ME-132.pdf\",\"contentType\":1}],\"stcMaximumPowerPmax\":655.000000000,\"stcVoltageAtMaximumPowerVmpp\":37.600000000,\"stcShortCircuitCurrentIsc\":18.380000000,\"stcOpenCircuitVoltageVoc\":45.700000000,\"stcCurrentAtMaximumPowerImpp\":17.420000000,\"temperatureCoefficientOfPmax\":-0.340000000,\"temperatureCoefficientOfVoc\":-0.250000000,\"temperatureCoefficientOfIsc\":0.040000000,\"firstYearPowerDegradation\":0E-9,\"subsequentAnnualPowerDegradation\":0.570000000,\"height\":2384.000,\"width\":1303.000,\"depth\":35.000,\"weight\":33.000000000,\"frameColor\":\"\",\"stcPanelEfficiency\":21.100000000,\"noctMaximumPowerPmax\":492.000000000,\"noctVoltageAtMaximumPowerVmpp\":35.300000000,\"noctCurrentAtMaximumPowerImpp\":13.940000000,\"noctOpenCircuitVoltageVoc\":42.600000000,\"noctShortCircuitCurrentIsc\":14.700000000,\"connectorType\":\"\"}",
          "quantity": 6,
          "unitCost": 90,
          "unitPrice": 100,
          "margin": 10,
          "lineCost": 540,
          "linePrice": 600,
          "materialsChangedStatus": 1,
          "proposalVisibleValues": null
      }
  ])
  setBosList([
    {
      "id": 667,
      "quoteId": 1225,
      "companyId": 86,
      "itemId": 885,
      "itemName": "AC Cable Run",
      "unit": "meter",
      "quantity": 2,
      "unitCost": 60,
      "unitPrice": 100,
      "margin": 40,
      "lineCost": 120,
      "linePrice": 200,
      "sort": 0
    },
    {
      "id": 668,
      "quoteId": 1225,
      "companyId": 86,
      "itemId": 2299,
      "itemName": "sdfdsf",
      "unit": "",
      "quantity": 3,
      "unitCost": 50,
      "unitPrice": 100,
      "margin": 50,
      "lineCost": 150,
      "linePrice": 300,
      "sort": 0
    },
  ])
  setAcList([
    {
      "id": 359,
      "quoteId": 1225,
      "companyId": 86,
      "itemId": 1878,
      "itemName": "Installation Cost",
      "unit": "job",
      "quantity": 2,
      "unitCost": null,
      "unitPrice": 60,
      "margin": null,
      "lineCost": null,
      "linePrice": 120,
      "sort": 0
    },
    {
      "id": 401,
      "quoteId": 1225,
      "companyId": 86,
      "itemId": 1893,
      "itemName": "Scissor Lift Hire",
      "unit": "watt_hour",
      "quantity": 1,
      "unitCost": 60,
      "unitPrice": 80,
      "margin": 25,
      "lineCost": 60,
      "linePrice": 80,
      "sort": 0
    }
  ])
  setCustomDeductionList([{
    "id": 31,
    "quoteId": 1225,
    "companyId": 86,
    "itemName": "Custom Deduction Item",
    "unitPrice": 1000,
    "linePrice": 1000,
  }])

  // setDefaultDeductions({
  //   companyId: 86,
  //   country: 'AU',
  //   id: 33,
  //   quoteId: 1159,
  //   deduction: `{"supportStc":1,"stcSwitch":1,"stcGstType":"gst_free","stcQuantity":1,"supportVic":1,"vicSwitch":1,"vicGstType":"gst_free","supportVicPv":1,"vicPvGstType":"gst_free","vicPvSwitch":1,"supportStcBattery":1,"stcBatterySwitch":1,"stcBatteryGstType":"gst_free","stcBatteryQuantity":1,"supportBess2":1,"bess2Switch":1,"bess2GstType":"gst_free","bess2Quantity":2,"vicPvVisible":1,"vicPvTotalPrice":100}`,
  // })
}

onMounted(() => {
  initData()
})

const cacheData = ref({})

watch([
  () => finalGroup.value,
  () => billInfo.value,
], vals => {
  cacheData.value = CalcInst.queryCacheStat()
}, { immediate: true, deep: true })

const isFetching = ref(false)
</script>

<style lang="scss" scoped>
.item-row {
  @apply relative flex text-sm justify-between items-center mb-2;
}
.auto-item {
  @apply flex items-center justify-end items-center text-xs text-gray;
}

:deep(.user-input-area) {
  input {
    text-align: right;
  }
}
</style>
