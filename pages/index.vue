<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-row>
        <v-col>
          <v-card>
            <v-card-title class="headline">
              Welcome to the Gutendex API query thing
            </v-card-title>
            <v-card-text>
              <p>
                This is a project to try out Nuxt and Vuetify. It's a simple app that allows you to query the Gutendex API.
              </p>
              <p>
                For more information on the Gutendex API, see the
                <a
                  href="https://gutendex.com/"
                  target="_blank"
                  rel="noopener noreferrer"
                >
                  documentation </a
                >.
              </p>
            </v-card-text>
            <v-card-actions>
              <v-spacer />
              <v-btn color="primary" nuxt to="/books"> Continue </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'IndexPage',
}
</script>
