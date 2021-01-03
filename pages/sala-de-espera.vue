<template>
    <CtCard title="Sala de espera" width="300" class="mx-auto">
      <v-row dense>
        <v-col cols="12" class="mt-5">
          En breves momentos serás redirigido hacia la página de <span v-if="user && userRoles.includes('admin')">pagos</span><span v-else>home</span>.
        </v-col>
        <v-col cols="12" class="mt-5">
          <v-row>
            <v-spacer />
            <v-subheader v-html="'¿Ha pasado más de 5 segundos?'" />
            <v-spacer />
          </v-row>
        </v-col>
        <v-col cols="12" class="my-5">
          <template v-if="user && userRoles.includes('admin')">
            <CtBtn to="/admin/payment" type="accent" block>
              Ir a pagos
            </CtBtn>
          </template>
          <template v-else>
            <CtBtn to="/" type="accent" block>
              Ir a home
            </CtBtn>
          </template>
        </v-col>
      </v-row>
    </CtCard>
</template>

<script>
export default {
  mounted() {
    setTimeout(() => (this.user && this.userRoles.includes('admin')) ? this.$router.push({ path: '/admin/payment' }) : this.$router.push({ path: '/' }), 3000)
  },

  computed: {
    user() {
      return this.$store.state.user.user
    },

    userRoles () {
      return (this.$store.state.user.user) ? this.$store.state.user.user.role.split(',') : []
    },
  },
}
</script>
