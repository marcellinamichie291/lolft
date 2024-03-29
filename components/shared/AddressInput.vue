<template>
  <div>
    <b-field :type="type" :message="err" :label="$t(label)">
      <b-input ref="address" v-model="inputValue" @input="handleInput" />
    </b-field>
  </div>
</template>

<script lang="ts">
import correctFormat from '@/utils/ss58Format'
import { checkAddress, isAddress } from '@polkadot/util-crypto'
import { Debounce } from 'vue-debounce-decorator'
import { Component, Emit, Prop, Ref, Vue } from 'nuxt-property-decorator'

@Component({})
export default class AddressInput extends Vue {
  @Prop(String) public value!: string
  private err: string | null = ''
  @Prop({ type: String, default: 'insert address' }) public label!: string
  @Prop(Boolean) public emptyOnError!: boolean
  @Prop({ type: Boolean, default: true }) public strict!: boolean

  @Ref('address') readonly address

  get inputValue(): string {
    return this.value
  }

  set inputValue(value: string) {
    this.handleInput(value)
  }

  get type(): string {
    return this.err ? 'is-danger' : ''
  }

  public focusInput(): void {
    this.address?.focus()
  }

  @Debounce(500)
  @Emit('input')
  protected handleInput(value: string) {
    if (this.strict) {
      const [, err] = checkAddress(value, correctFormat(this.ss58Format))
      this.err = value ? err : ''
    } else {
      this.err = isAddress(value) ? '' : 'Invalid address'
    }

    return this.emptyOnError && this.err ? '' : value
  }

  get ss58Format(): number {
    return this.$store.getters['chain/getChainProperties58Format']
  }
}
</script>
