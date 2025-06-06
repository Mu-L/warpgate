<script lang="ts">
    import {
        Button,
        Form,
        FormGroup,
        Input,
        Modal,
        ModalBody,
        ModalFooter,
    } from '@sveltestrap/sveltestrap'

    import { type ExistingPublicKeyCredential } from './lib/api'

    interface Props {
        isOpen: boolean
        instance?: ExistingPublicKeyCredential
        save: (label: string, opensshPublicKey: string) => void
    }

    let {
        isOpen = $bindable(true),
        instance,
        save,
    }: Props = $props()

    let field: HTMLInputElement|undefined = $state()
    let label: string = $state('')
    let opensshPublicKey: string = $state('')
    let validated = $state(false)

    const PK_REGEX = /^ssh-([\w-]+) [A-Za-z0-9+/=]+( (?<comment>[^ ]+))?$/

    function _save () {
        if (!opensshPublicKey || !label) {
            return
        }
        if (opensshPublicKey.includes(' ')) {
            const parts = opensshPublicKey.split(' ').filter(x => x)
            opensshPublicKey = `${parts[0]} ${parts[1]}`
        }
        isOpen = false
        save(label, opensshPublicKey)
    }

    function _cancel () {
        isOpen = false
    }

    $effect(() => field?.addEventListener('paste', e => {
        const clipboardData = e.clipboardData
        if (clipboardData) {
            const newValue = clipboardData.getData('text')
            onPublicKeyPaste(newValue)
        }
    }))

    function onPublicKeyPaste (newValue: string) {
        const match = PK_REGEX.exec(newValue)
        if (!label && match) {
            label = match.groups?.comment || ''
        }
    }
</script>

<Modal toggle={_cancel} isOpen={isOpen} on:open={() => {
    if (instance) {
        label = instance.label
        opensshPublicKey = instance.opensshPublicKey
    }
    field?.focus()
}}>
    <Form {validated} on:submit={e => {
        _save()
        e.preventDefault()
    }}>
        <ModalBody>
            <FormGroup floating label="Label">
                <Input
                    bind:inner={field}
                    type="text"
                    required
                    bind:value={label} />
            </FormGroup>
            <FormGroup floating label="Public key in OpenSSH format" spacing="0">
                <Input
                    style="font-family: monospace; height: 15rem"
                    bind:inner={field}
                    type="textarea"
                    required
                    placeholder="ssh-XXX YYYYYY"
                    bind:value={opensshPublicKey} />
            </FormGroup>
        </ModalBody>
        <ModalFooter>
            <Button
                type="submit"
                color="primary"
                class="modal-button"
                on:click={() => validated = true}
            >Save</Button>

            <Button
                class="modal-button"
                color="danger"
                on:click={_cancel}
            >Cancel</Button>
        </ModalFooter>
    </Form>
</Modal>
