<template>
    <header>
        <a class="header"><img src="https://www.avito.st/s/cc/resources/35f5a0d67b53.svg" class="logo"></a>
    </header>
</template>

<script>
export default {
    
}
</script>

<style scoped>
header {
    display: flex;
    align-items: center;
}

.header {
    padding: 10px;
    text-align: center;
    cursor: pointer;
    font-family: Ubuntu;
}

.logo {
    display: inline-block;
    width: 100px;
}

.header:hover {
    color: blue;
}
</style>
