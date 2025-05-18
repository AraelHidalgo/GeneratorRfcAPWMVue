<script setup>
import { ref, computed } from 'vue';

// Form data
const nombre = ref('');
const apellidoPaterno = ref('');
const apellidoMaterno = ref('');
const fechaNacimiento = ref(null);
const rfc = ref('');
const submitted = ref(false);
const generationSuccess = ref(false);

// Function to check if a character is a vowel
const isVowel = (char) => {
    const vowels = ['A', 'E', 'I', 'O', 'U'];
    return vowels.includes(char.toUpperCase());
};

// Function to get the first vowel after the first character in a string
const getFirstVowelAfterFirst = (str) => {
    if (!str || str.length < 2) return '';
    for (let i = 1; i < str.length; i++) {
        if (isVowel(str[i])) {
            return str[i].toUpperCase();
        }
    }
    return 'X'; // If no vowel is found, use X as default
};

// Function to generate RFC
const generateRfc = () => {
    submitted.value = true;

    // Validate all fields are filled
    if (!nombre.value || !apellidoPaterno.value || !apellidoMaterno.value || !fechaNacimiento.value) {
        return;
    }

    try {
        // Clean and normalize strings
        const nombreClean = nombre.value.trim().toUpperCase();
        const paternoClean = apellidoPaterno.value.trim().toUpperCase();
        const maternoClean = apellidoMaterno.value.trim().toUpperCase();

        // Get first part of RFC: First letter of paternal surname + First vowel after first letter + First letter of maternal surname + First letter of first name
        let rfcLetters = '';

        // First letter of paternal surname
        rfcLetters += paternoClean.charAt(0);

        // First vowel after first letter of paternal surname
        rfcLetters += getFirstVowelAfterFirst(paternoClean);

        // First letter of maternal surname (or X if not available)
        rfcLetters += maternoClean.charAt(0) || 'X';

        // First letter of first name
        rfcLetters += nombreClean.charAt(0);

        // Format date as YYMMDD
        const date = new Date(fechaNacimiento.value);
        const year = date.getFullYear().toString().slice(2);
        // Month is 0-indexed, so add 1 and ensure it's padded with a leading zero if needed
        const month = (date.getMonth() + 1).toString().padStart(2, '0');
        const day = date.getDate().toString().padStart(2, '0');

        // Combine letters and date
        const rfcBase = rfcLetters + year + month + day;

        // Add a simple homoclave (in reality, this is more complex and involves additional calculations)
        // For this example, we'll just add a random 3-character code
        const homoclave = 'ABC';

        // Set the RFC
        rfc.value = rfcBase + homoclave;
        generationSuccess.value = true;
    } catch (error) {
        console.error('Error generating RFC:', error);
        rfc.value = 'Error al generar RFC';
        generationSuccess.value = false;
    }
};

// Reset form
const resetForm = () => {
    nombre.value = '';
    apellidoPaterno.value = '';
    apellidoMaterno.value = '';
    fechaNacimiento.value = null;
    rfc.value = '';
    submitted.value = false;
    generationSuccess.value = false;
};

// Compute if form is valid
const isFormValid = computed(() => {
    return nombre.value.trim() && apellidoPaterno.value.trim() && apellidoMaterno.value.trim() && fechaNacimiento.value;
});
</script>

<template>
    <div class="card">
        <h5>Generador de RFC</h5>
        <div class="form-vertical">
            <div class="form-group">
                <label for="nombre" class="font-bold">Nombre(s)</label>
                <InputText id="nombre" v-model="nombre" :class="{ 'p-invalid': submitted && !nombre.trim() }" aria-describedby="nombre-error" />
                <small id="nombre-error" class="p-error" v-if="submitted && !nombre.trim()">El nombre es requerido.</small>
            </div>

            <div class="form-group">
                <label for="apellidoPaterno" class="font-bold">Apellido Paterno</label>
                <InputText id="apellidoPaterno" v-model="apellidoPaterno" :class="{ 'p-invalid': submitted && !apellidoPaterno.trim() }" aria-describedby="paterno-error" />
                <small id="paterno-error" class="p-error" v-if="submitted && !apellidoPaterno.trim()">El apellido paterno es requerido.</small>
            </div>

            <div class="form-group">
                <label for="apellidoMaterno" class="font-bold">Apellido Materno</label>
                <InputText id="apellidoMaterno" v-model="apellidoMaterno" :class="{ 'p-invalid': submitted && !apellidoMaterno.trim() }" aria-describedby="materno-error" />
                <small id="materno-error" class="p-error" v-if="submitted && !apellidoMaterno.trim()">El apellido materno es requerido.</small>
            </div>

            <div class="form-group">
                <label for="fechaNacimiento" class="font-bold">Fecha de Nacimiento</label>
                <Calendar id="fechaNacimiento" v-model="fechaNacimiento" dateFormat="dd/mm/yy" :showIcon="true" :class="{ 'p-invalid': submitted && !fechaNacimiento }" aria-describedby="fecha-error" />
                <small id="fecha-error" class="p-error" v-if="submitted && !fechaNacimiento">La fecha de nacimiento es requerida.</small>
            </div>

            <div class="form-actions">
                <Button label="Generar RFC" icon="pi pi-check" @click="generateRfc" :disabled="!isFormValid" />
                <Button label="Limpiar" icon="pi pi-times" class="p-button-secondary" @click="resetForm" />
            </div>

            <div v-if="rfc" class="p-4 border-round bg-primary-50 mb-3">
                <div class="text-xl font-bold mb-2">RFC Generado:</div>
                <div class="text-2xl font-bold">{{ rfc }}</div>
                <small class="block mt-2 text-sm"> Nota: Este RFC es generado con fines demostrativos. La homoclave real requiere cálculos adicionales y puede variar. </small>
            </div>
        </div>
    </div>
</template>

<style scoped>
.card {
    background: var(--surface-card);
    padding: 2rem;
    border-radius: 10px;
    margin-bottom: 1rem;
    box-shadow: var(--card-shadow);
}

.form-vertical {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
}

.form-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.form-actions {
    display: flex;
    flex-direction: row;
    gap: 1rem;
}

.p-invalid {
    border-color: var(--red-500);
}

.p-error {
    color: var(--red-500);
}
</style>
