<template>
    <div class="hotlead-presentational mvt-hotleadform grid grid-xs-1 large" :class="{'confirming': showConfirmUI}">
        <div v-if="profile" class="dialog-profile">
            <div v-if="options.profileHighlighter" class="highlighter"></div>
            <i v-if="/^icon-/.test(profile)" class="circle default huge" :class="profile" />
            <mvtMapStatic v-else-if="showStreetView" class="img" :cacheFirst="true" :lat="propertyData.geo.lat" :lng="propertyData.geo.lng" type="streetview" streetViewDefautType="satellite" :address="propertyData.geo.address"  width="70"  height="70"></mvtMapStatic>
            <img v-else :src="profile" class="circle default huge" />
        </div>

        <div class="mvt-hotlead-confirm">
            <div class="grid grid-xs-1">
                <div class="flex middle center left f3"><i class="icon-send"></i> Sending message...</div>
                <div class="skeleton"></div>
            </div>
        </div>

        <div class="text-center">
            <div class="f3 m-t-3 m-b-3" v-once v-if="showConfirmTitle">Success! We will be in touch shortly.</div>
            <h3 class="hotlead-title f3 text-bold" data-role="title" v-show="title" v-html="title"></h3>
            <div v-if="subtitle" v-html="subtitle"></div>
        </div>
        <form
            class="form form-default grid grid-xs-1 large"
            :class="formClass"
            v-submit="submitForm"
            novalidate
        >
            <div v-if="hasOJOAgent && leadForm && leadForm.showOJOagentInfo" class="assigned-agent-info text-center">
                <a class="link phone" :href="'tel:' + agentPhone">
                    <i class="icon-phone-o"></i>{{ agentPhone }}
                </a>
            </div>
            <mvtCalendar v-if="options.enableChangeDate" :value="localCalendar" @input="updateCalendar" />
            <template v-if="hasOJOAgent">
                <div class="assigned-agent">
                    <div class="cta-section" v-if="leadForm && leadForm.showCtaLinks">
                        <div class="cta-sub">I'd like to..</div>
                        <div class="cta-links">
                            <a v-for="(item,i) in assignedAgentCtaLinks" :key="i"
                                @click="changeActiveCTA(i, item.value)"
                                :class="{active:activeCTAIndex == i}">
                                    {{item.text}}
                            </a>
                        </div>
                    </div>
                    <mvtTextbox class="message">
                        <div class="textarea">
                            <label>{{leadForm && leadForm.commnetLabel}}</label>
                            <textarea name="comment" v-model="hotlead.comment" maxlength="4000"></textarea>
                        </div>
                    </mvtTextbox>
                </div>

                <mvtTextbox v-if="showAddress">
                    <mvtInputGeo :model="addressInput" :isHotleadField="true" :hideButton="true" v-validate="'required|geo'"  data-geotype="zipcode" :placeholder="'Address'"  :options="{types:['address']}"/>
                </mvtTextbox>
                <mvtTextbox v-else-if="showZipcode">
                    <mvtInputGeo :model="addressInput" :isHotleadField="true" :hideButton="true" v-validate="'required|geo'" data-geotype="zipcode" :placeholder="'ZIP Code'" :options="{types:['postal_code']}"/>
                </mvtTextbox>
                <mvtTextbox v-else-if="showInputGeo">
                    <mvtInputGeo :model="addressInput" :isHotleadField="true" :hideButton="true" v-validate="'required|geo'" data-geotype="zipcode" :placeholder="'Address or ZIP Code'"/>
                </mvtTextbox>

                <div class="form-input" v-if="!hasDefaultPhone">
                    <mvtTextbox>
                        <div class="input">
                            <label>{{ labels.phoneInput }}</label>
                            <input
                                ref="phone"
                                type="tel"
                                name="phone"
                                v-model="hotlead.phone"
                                v-inputformat:phone
                                :autocomplete="isACForm ? 'off': 'tel-national'"
                                v-validate="'required|phoneNumber'"
                                maxlength="12"
                                placeholder="123-123-1234"
                            />
                        </div>
                    </mvtTextbox>
                </div>
            </template>
            <div class="form-input" v-else>
                <mvtSelect class="input" v-if="enquiryInput && !isACForm" ref="enquiryTypes" v-model="hotlead.enquiryType" :data="enquiryData" v-validate="'required|selectOption'"/>
                <mvtRadio v-if="isACForm" class="flex nowrap center" :model="acFormLeadType" :data="leadChecklist"/>
                <mvtTextbox  v-if="!glb.user.fullName || editFullName">
                <div :class="{'textbox': !hotlead.name, 'textbox focus': hotlead.name}">
                    <div class="input">
                        <label>{{ labels.name }}</label>
                        <input
                            ref="name"
                            type="text"
                            name="name"
                            v-inputtrim
                            v-model="hotlead.name"
                            maxlength="128"
                            v-validate="'required|name'"
                            :autocomplete="isACForm ? 'off': 'name'"
                            placeholder="Name" validation>
                    </div>
                  </div>
                </mvtTextbox>
                <mvtTextbox v-if="!glb.user.email || editEmail">
                    <div :class="{'textbox': !hotlead.email, 'textbox focus': hotlead.email}">
                        <div class="input">
                        <label>{{ labels.email }}</label>
                        <input
                            type="email"
                            name="email"
                            v-model="hotlead.email"
                            maxlength="128"
                            :autocomplete="isACForm ? 'off': 'email'"
                            v-inputtrim
                            v-validate="'required|email'"
                            placeholder="Email" validation>
                        </div>
                    </div>
                </mvtTextbox>
                <mvtTextbox v-if="!glb.user.phones || editPhone">
                    <div :class="{'textbox': !hotlead.phone, 'textbox focus': hotlead.phone}">
                        <div class="input">
                        <label>{{ labels.phoneInput }}</label>
                        <input
                            ref="phone"
                            type="tel"
                            name="phone"
                            v-model="hotlead.phone"
                            v-inputformat:phone
                            :autocomplete="isACForm ? 'off': 'tel-national'"
                            v-validate="'required|phoneNumber'"
                            maxlength="12"
                            placeholder="Mobile Phone" validation>
                    </div>
                  </div>
                </mvtTextbox>
                <mvtTextbox  v-if="options.propertyData && options.propertyData.isRentals">
                    <div class="input right">
                        <label>{{ moveDate }}</label>
                        <mvtInputDate v-if="moveDate" ref="inputDate" v-model="hotlead.moveIn" :min="moveDate" v-validate="'required'" :disableWeekends="false" type="date" />
                    </div>
                </mvtTextbox>
                <mvtTextbox v-if="showAddress">
                    <mvtInputGeo :model="addressInput" :isHotleadField="true" :hideButton="true" v-validate="'required|geo'"  data-geotype="zipcode" :placeholder="'Address'"  :options="{types:['address']}"/>
                </mvtTextbox>
                <mvtTextbox v-else-if="showZipcode">
                    <mvtInputGeo  :model="addressInput" :isHotleadField="true" :hideButton="true" v-validate="'required|geo'" data-geotype="zipcode" :placeholder="'ZIP Code'" :options="{types:['postal_code']}"/>
                </mvtTextbox>
                <mvtTextbox v-else-if="showInputGeo">
                    <mvtInputGeo :model="addressInput" :isHotleadField="true" :hideButton="true" v-validate="'required|geo'" data-geotype="zipcode" :placeholder="'Address or ZIP Code'"/>
                </mvtTextbox>
                <mvtTextbox v-if="forceInputPrice">
                    <div class="input">
                        <label>Price</label>
                        <mvtInputNumber
                            name="price"
                            v-model="hotlead.price"
                            maxlength="12"
                            :autocomplete="isACForm ? 'off': 'price'"
                            v-validate="'required|amount'"
                            v-inputtrim
                            :prefix="'$'"
                        />
                    </div>
                </mvtTextbox>

                <mvtTextbox class="comments" v-if="showComment">
                    <div class="textarea">
                        <label v-if="messageAgentLeadType">{{ labels.buySellComments }}</label>
                        <label v-else>{{ labels.comments }}</label>
                        <textarea name="comment" v-model="hotlead.comment" maxlength="4000" v-validate="'required|comment'"></textarea>
                    </div>
                </mvtTextbox>
                <div v-if="isACForm" class="text-danger">{{errorMessage}}</div>
            </div>
            <template v-if="showVeteransOption">
            <mvtCheckbox
                :class="getSplit('movoto-veteran-copy-size-CW-11236') ? 'large singleline' : 'small'"
                v-model="localIsVeteran"
                @input="updateIsVeteran"
                :data="getVeteranCheckboxData"
            ></mvtCheckbox>
                <mvtSelect v-show="showVeteransType" :data="veteransTypeData"  v-model="hotlead.veteransType" @change="selectVeteranType" />
            </template>
            <div class="text-error f7" v-if="errorMsg" v-html="errorMsg"></div>

            <button @click="submitForm" class="large" :class="options.buttonClass || 'btn main active'" type="submit" :disabled="disableSend" >{{ buttonText }} </button>
            <div class="flex middle center f7" v-if="hasOJOAgent || showConcierge">
                <span class="f4 icon-live-connect"></span> Need help?
                <a class="link" @click="openRequestAssistance">Contact your Concierge</a>
            </div>
            <div class="text-a f8 lh-normal" v-if="hasOJOAgent && leadForm && leadForm.showDisclaimer">
                {{ labels.ojoLeadFormDisclaimer }}
            </div>
            <div class="text-a f8 lh-normal" v-show="visiableDisclaimer" v-if="leadDisclaimer" v-html="leadDisclaimer"></div>
        </form>
        <div class="disclaimer-tootip" v-if="popDialogBox">
                <div @click="closeTip" class="disclaimer-tootip-overlay"></div>
                <div class="disclaimer-tootip-content">
                    <a @click="closeTip"><i class="icon-times"></i></a>
                    <div>
                        {{`Bankrate, LLC will be seeking to connect you with up to four (4) Lending Partners and is a mortgage licensee – #1743443 (<a class="text-dotted" href="https://nam02.safelinks.protection.outlook.com/?url=http%3A%2F%2Fwww.nmlsconsumeraccess.org%2F&data=05%7C01%7Cjlopes%40redventures.com%7C5e5047e0c6d7470cb37e08da68c54799%7C4289d6102cfd46218c9644a1518ddb0a%7C0%7C0%7C637937493777787408%7CUnknown%7CTWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwiLCJXVCI6Mn0%3D%7C3000%7C%7C%7C&sdata=AsqCcHC1qxgJe7SUAGWWOlZ0SdYa%2FzsCV%2FeZ7F7Pzh4%3D&reserved=0" target="_blank" ref="nofollow noreferrer">www.nmlsconsumeraccess.org</a>) | <a class="text-dotted" href="https://www.bankrate.com/privacy/" target="_blank" ref="nofollow noreferrer">Privacy Policy</a> | <a class="text-dotted" href="https://www.bankrate.com/terms/" target="_blank" ref="nofollow noreferrer">Terms of Use</a> | <a class="text-dotted" href="https://www.bankrate.com/licenses/" target="_blank" ref="nofollow noreferrer">Licensing Information</a>  | Equal Housing Opportunity. Lending Partners who contact you are not recommended or endorsed by Bankrate, LLC, are not the only providers of mortgage loan services of the kinds they offer and typically will have paid for the opportunity to contact you as a result of this advertisement to further offer their mortgage loan help and services. Standard mobile, message, or data rates may apply.  Qualifying for pre-approval or a loan is not guaranteed, and further information and actions may be required of you.`}}
                    </div>
                </div>
            </div>
        </div>
</template>

<script>
import mvtCalendar from '@/common/components/mvtcalendar/mvt-calendar.vue';
import mvtCheckbox from '@/common/components/mvtcheckbox/mvt-checkbox.vue';
import mvtInputDate from '@/common/components/mvtinputdate/mvt-inputdate.vue';
import mvtInputGeo from '@/common/components/mvtinputgeo/mvt-inputgeo.vue';
import mvtInputNumber from '@/common/components/mvtinputnumber/mvt-inputnumber.vue';
import mvtRadio from '@/common/components/mvtradio/mvt-radio.vue';
import mvtSelect from '@/common/components/mvtselect/mvt-select.vue';
import mvtTextbox from '@/common/components/mvttextbox/mvt-textbox.vue';
import mvtMapStatic from '@/common/components/mvtmapstatic/mvt-mapstatic.vue';

export default {
    name: 'HotLeadFormPresentational',
    props: {
        profile: String,
        hotlead: Object,
        options: Object,
        title: String,
        subtitle: String,
        formClass: String,
        submitForm: Function,
        hasOJOAgent: Boolean,
        leadForm: Object,
        agentPhone: String,
        enquiryInput: Boolean,
        enquiryData: Array,
        leadChecklist: Array,
        moveDate: String,
        errorMessage: String,
        assignedAgentCtaLinks: Array,
        activeCTAIndex: Number,
        hasDefaultPhone: Boolean,
        isACForm: Boolean,
        glb: Object,
        editFullName: Boolean,
        editEmail: Boolean,
        editPhone: Boolean,
        showAddress: Boolean,
        showZipcode: Boolean,
        showInputGeo: Boolean,
        showStreetView: Boolean,
        showConfirmUI: Boolean,
        showConfirmTitle: Boolean,
        propertyData: Object,
        forceInputPrice: Boolean,
        showComment: Boolean,
        isVeteran: Boolean,
        getVeteranCheckboxData: Object,
        showVeteransType: Boolean,
        veteransTypeData: Array,
        visiableDisclaimer: Boolean,
        leadDisclaimer: String,
        popDialogBox: Boolean,
        buttonText: String,
        formattedPhone: String,
        labels: Object,
        showConcierge: Boolean,
        showVeteransOption: Boolean,
        errorMsg: String,
        disableSend: Boolean,
        addressInput: Object,
        localCalendar: String,
        updateCalendar: Function,
        acFormLeadType: String,
    },
    components: {
        mvtCalendar,
        mvtCheckbox,
        mvtInputDate,
        mvtInputGeo,
        mvtInputNumber,
        mvtTextbox,
        mvtSelect,
        mvtRadio,
        mvtMapStatic,
   },
};
</script>
