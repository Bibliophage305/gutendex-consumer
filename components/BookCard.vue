<template>
  <v-card>
    <v-card-title style="word-break: break-word;">
      {{ book.title }}
    </v-card-title>
    <v-card-subtitle style="word-break: break-word;">
      {{ book.authors.map(author => author.name).join('; ') }}
    </v-card-subtitle>
    <v-card-text>
      Downloaded {{ book.download_count }} times
    </v-card-text>
    <v-card-actions>
      <v-spacer />
      <v-btn color="primary" nuxt :to="`/books/${book.id}`">
        More
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
export default {
  name: 'BookCard',

  props: {
    book: {
      type: Object,
      required: true,
    },
  },
}
</script>
