<template>
    <modal ref="modal" title="Select Token" class="modal_main">
        <div class="token_select_body">
            <div class="list">
                <div class="token_row" @click="select('native')">
                    <img src="/img/avax_icon_circle.png" class="col_img" />
                    <div class="col_name">
                        <p>AVAX</p>
                        <p>Avalanche</p>
                    </div>
                    <p class="col_bal">{{ avaxBalance.toLocaleString() }}</p>
                </div>
                <div v-for="t in tokens" :key="t.data.address" class="token_row" @click="select(t)">
                    <img v-if="t.data.logoURI" :src="t.data.logoURI" class="col_img" />
                    <p v-else class="col_img">?</p>
                    <div class="col_name">
                        <p>{{ t.data.symbol }}</p>
                        <p>{{ t.data.name }}</p>
                    </div>
                    <p class="col_bal">{{ t.balanceBig.toLocaleString() }}</p>
                </div>
            </div>
        </div>
    </modal>
</template>
<script lang="ts">
import 'reflect-metadata'
import { Vue, Component, Prop } from 'vue-property-decorator'

import Modal from '@/components/modals/Modal.vue'
import Erc20Token from '@/js/Erc20Token'
import Big from 'big.js'
import { WalletType } from '@/store/types'
import { bnToBig } from '@/helpers/helper'

@Component({
    components: {
        Modal,
    },
})
export default class EVMTokenSelectModal extends Vue {
    $refs!: {
        modal: Modal
    }
    open(): void {
        let modal = this.$refs.modal as Modal
        modal.open()
    }

    get tokens(): Erc20Token[] {
        let tokens: Erc20Token[] = this.$store.getters['Assets/networkErc20Tokens']
        let filt = tokens.filter((t) => {
            if (t.balanceBN.isZero()) return false
            return true
        })
        return filt
    }

    // get symbol() {
    //     if (this.selected === 'native') return 'AVAX'
    //     else return this.selected.data.symbol
    // }

    get avaxBalance(): Big {
        let w: WalletType | null = this.$store.state.activeWallet
        if (!w) return Big(0)
        let balBN = w.ethBalance
        return bnToBig(balBN, 18)
    }

    select(token: Erc20Token | 'native') {
        // this.selected = token
        this.$emit('select', token)
        this.close()
    }

    close() {
        this.$refs.modal.close()
    }
}
</script>
<style scoped lang="scss">
@use '../../../main';

.token_select_body {
    width: 420px;
    max-width: 100%;
    //padding: 10px 20px;
}

.list {
    //position: absolute;
    //top: 0;
    //left: 100%;
    //width: 260px;
    //max-height: 0px;
    max-height: 70vh;
    overflow: scroll;
    z-index: 2;
    border-radius: 4px;
}

$logo_w: 38px;

.token_row {
    font-size: 15px;
    padding: 10px 20px;
    display: grid;
    grid-template-columns: max-content max-content 1fr;
    column-gap: 12px;
    cursor: pointer;
    user-select: none;

    > * {
        align-self: center;
    }

    img {
        object-fit: contain;
    }

    &:hover {
        background-color: var(--bg-light);

        .col_img {
            background-color: var(--primary-color);
            color: var(--bg-wallet);
        }
    }
}

.col_img {
    width: $logo_w;
    height: $logo_w;
    border-radius: $logo_w;
    background-color: var(--bg-light);
    text-align: center;
    line-height: $logo_w;
}

.col_bal {
    text-align: right;
}

.col_name {
    p:last-of-type {
        font-size: 13px;
        color: var(--primary-color-light);
    }
}

@include main.mobile-device {
    .token_select_body {
        width: 100%;
        height: 40vh;
    }
}
</style>
