<template>
    <div>
        <div v-for="voucher in vouchers" v-bind:key="voucher.id" class="card-pf" id="voucher-coupon">
            <img class="voucher-logo" src="/static/logo.png" />
            <h2 class="card-pf-title voucher-name" id="test">
                {{info.data.name}}
            </h2>
            <h3 class="card-pf-title voucher-desc">
                {{info.data.description}}
            </h3>
            <div class="card-pf-body voucher-main">
                <p>
                    <strong class="voucher-code">{{voucher.code}}</strong>
                </p>
            </div>
            <div class="card-pf-footer voucher-details">
                <div class="card-pf-time-frame-filter voucher-valid">
                    Valid:
                    <strong>{{voucher.duration}}</strong>
                </div>
                <div class="voucher-download">
                    <span class="fa fa-arrow-down"></span> Download:
                    <strong>{{voucher.bandwidth_down}}</strong>
                </div>
                <div class="voucher-upload">
                    <span class="fa fa-arrow-up"></span> Upload:
                    <strong>{{voucher.bandwidth_up}}</strong>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'Voucher',
        props: {
            vouchers: {
                type: Object
            }
        }
    }
</script>

<style scoped>
    #voucher-coupon {
        display: none;
    }
    .voucher-logo {
        height: 25px;
        margin: 15px;
        margin-left: -5px;
    }
    .voucher-name {
        position: absolute;
        top: -8px;
        left: 90px;
        max-width: 200px;
        color: goldenrod;
    }
    .voucher-desc {
        position: absolute;
        top: 15px;
        left: 90px;
        font-size: 14px;
        color: #757575;
        max-width: 200px;
    }
    .voucher-code {
        font-size: 30px;
        margin-top: 5px;
    }
    .voucher-valid {
        font-size: 15px;
        margin-top: 6px;
    }
    .voucher-main {
        text-align: center;
        padding-bottom: 10px;
        margin-top: 15px;
    }
    .voucher-details {
        padding: 10px;
        background: #fff;
    }
</style>
