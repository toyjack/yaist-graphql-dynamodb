<script setup lang="ts">
const results = ref({})
const term = ref("一不")

const inputedNumber = computed(() => {
  const matchedNumber = term.value.match(/\d+/g)
  if (matchedNumber) {
    return parseInt(matchedNumber[0])
  } else {
    return 0
  }
})
const termIDS = computed(() => term.value.replace(/\d+/g, ''))

function intersection(arrs: any[][]) {
  let prev_arr: string[] = arrs[0]
  for (let arr of arrs) {
    prev_arr = prev_arr.filter((x) => arr.includes(x));
  }
  return prev_arr as string[];
} 

async function getStroke(hanzi: string) {
  const query = `query getChar {
  getStroke(key: "${hanzi}"){
    key
    value
  }
}`
  const { data } = await useFetch(() => 'https://wtmjjlcffnfozlalge26bo6xey.appsync-api.ap-northeast-1.amazonaws.com/graphql',
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': 'da2-4u56k7loszhn7j2b5cfr3vq32m'
      },
      body: JSON.stringify({
        query: query
      })
    })

  return data.value.data.getStroke.value as number
}

async function getStrokeMultiple(hanziArr: string[]) {
  let query = "query getStrokeMultiple {"
  let index = 1
  for (let hanzi of hanziArr) {
    query += `
  item${index.toString()}:getStroke(key: "${hanzi}"){
    key
    value
  }
`
    index += 1
  }
  query += "}"

  const { data } = await useFetch(() => 'https://wtmjjlcffnfozlalge26bo6xey.appsync-api.ap-northeast-1.amazonaws.com/graphql',
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': 'da2-4u56k7loszhn7j2b5cfr3vq32m'
      },
      body: JSON.stringify({
        query: query
      })
    })

  return data
}


async function getHanzi(ids: string) {
  const query = `query getChar {
  getCharacters(key: "${ids}"){
    key
    value
  }
}`
  const { data } = await useFetch(() => 'https://wtmjjlcffnfozlalge26bo6xey.appsync-api.ap-northeast-1.amazonaws.com/graphql',
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': 'da2-4u56k7loszhn7j2b5cfr3vq32m'
      },
      body: JSON.stringify({
        query: query
      })
    })

  return data.value.data.getCharacters.value
}
let termStrokeCount = ref(0)

async function search() {
  termStrokeCount.value = 0
  let temp = []

  for (const ids of termIDS.value.split('')) {
    const hanzi = await getHanzi(ids)
    const stroke = await getStroke(ids) as number
    termStrokeCount.value += stroke
    temp.push(hanzi)
  }

  const targeStrokeCount = inputedNumber.value + termStrokeCount.value
  console.log(targeStrokeCount)
  if (inputedNumber.value){
    const resultsWithStoke = await getStrokeMultiple(intersection(temp))
    temp = []

    for (let k in resultsWithStoke.value.data) {
      if( resultsWithStoke.value.data[k]){
        let hanzi = resultsWithStoke.value.data[k].key
        let stroke = resultsWithStoke.value.data[k].value
        if (stroke === targeStrokeCount) {
          temp.push(hanzi)
        }
      }
    }
    results.value = temp

  }else{
  results.value = intersection(temp)

  }
}
</script>
<template>
  <div>
    <h2>yaist - with graphql</h2>
    <div>
      <div>
        <input type="text" v-model="term" @keyup.enter="search()">
        <button @click="search()">Search</button>
      </div>

      <div>
        Inputed Stroke Count Number: {{ inputedNumber }}
      </div>

      <div>
        <span>Stroke Count of Terms:</span>
        {{ termStrokeCount }}
      </div>

      <div>
        <h4>Results Number: {{ results.length }}</h4>

        <div>
          <span v-for="item of results">
            {{ item }}
          </span>
        </div>
      </div>
    </div>

  </div>
</template>
