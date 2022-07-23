<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
			<v-card v-if="error !== undefined">
				<v-card-title>
					Error
				</v-card-title>
				<v-card-subtitle>
					{{ error }}
				</v-card-subtitle>
				<v-card-actions>
					<v-spacer/>
					<v-btn color="primary" nuxt to="/books">
						Back
					</v-btn>
				</v-card-actions>
			</v-card>
			<v-card v-else>
				<v-card-title style="word-break: break-word;">
					{{ book.title }}
				</v-card-title>
				<v-card-subtitle style="word-break: break-word;">
					{{ book.authors.map(author => author.name).join('; ') }}
				</v-card-subtitle>
				<v-card-text>
					<v-row>
						<v-col>
							<v-row align="center" justify="center">
								<v-col>
									<v-select
										:items="Object.keys(book.formats)"
										label="Download format"
										v-model="downloadFormat"
									/>
								</v-col>
								<v-col cols="auto">
									<v-btn
										:href="book.formats[downloadFormat]"
										target="_blank"
										rel="noopener noreferrer"
									>
										Download
									</v-btn>
								</v-col>
							</v-row>
							<p>
								Downloaded {{ book.download_count }} times
							</p>
							<p>
								This book is {{ book.copyright ? 'in' : 'out of' }} copyright
							</p>
						</v-col>
					</v-row>
					<v-expansion-panels>
						<v-expansion-panel v-for="field in ['subjects', 'bookshelves', 'languages', 'translators'].filter(field => book[field].length > 0)" :key="field">
							<v-expansion-panel-header>
								{{ field.charAt(0).toUpperCase() + field.slice(1) }}
							</v-expansion-panel-header>
							<v-expansion-panel-content>
								<v-list>
									<v-list-item v-for="item in book[field]" :key="item">
										<v-list-item-content>
											<v-list-item-subtitle>
												{{ item }}
											</v-list-item-subtitle>
										</v-list-item-content>
									</v-list-item>
								</v-list>
							</v-expansion-panel-content>
						</v-expansion-panel>
					</v-expansion-panels>
				</v-card-text>
				<v-card-actions>
					<v-spacer/>
					<v-btn color="primary" nuxt to="/books">
						Back
					</v-btn>
				</v-card-actions>
			</v-card>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'BookPage',

	async asyncData({ params }) {
		let url = new URL(`https://gutendex.com/books/${params.id}`);
    const data = await fetch(url).then(response => response.json());
		if (data.detail) {
			return {
				error: data.detail
			};
		}
		const book = data;
		const downloadFormat = book.formats && Object.keys(book.formats).length > 0 ? Object.keys(book.formats)[0] : null;
		return {book, downloadFormat};
	},
}
</script>
