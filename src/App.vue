<template>
	<table>
		<thead>
			<tr>
				<th>KEY</th>
				<th>CELL1</th>
				<th>CELL2</th>
				<th>CELL3</th>
				<th>CELL4</th>
				<th>CELL5</th>
				<th>CELL6</th>
				<th>CELL7</th>
				<th>CELL8</th>
				<th>CELL9</th>
			</tr>
		</thead>
		<tbody>
			<tr
			  v-for="(data, index) in newData" :key="index"
				ref="rows"
				@click="selectRow(index)">
				<td>
					<i 
					  class="bi bi-star-fill star"
						style="font-size: 20px; margin-right: 5px;"
						ref="star"
						@click.stop="selectStar(index)">
					</i>{{ data.key }}
				</td>
				<td>{{ data.cell1 }}</td>
				<td>{{ data.cell2 }}</td>
				<td>{{ data.cell3 }}</td>
				<td>{{ data.cell4 }}</td>
				<td>{{ data.cell5 }}</td>
				<td>{{ data.cell6 }}</td>
				<td>{{ data.cell7 }}</td>
				<td>{{ data.cell8 }}</td>
				<td>{{ data.cell9 }}</td>
			</tr>
		</tbody>
	</table>
</template>

<script>
export default {
	data() {
		return {
			data1: [],
			data2: [],
			data3: [],
			newData: [],
			selectedRow: null,
			selectedStars: []
		};
	},
	beforeMount() {
	},
	mounted() {
		// 同時取得所有資料
		Promise.all([
			fetch('../data/data1.json').then(response => response.json()),
			fetch('../data/data2.json').then(response => response.json()),
			fetch('../data/data3.json').then(response => response.json())
		]).then(dataArray => {
			  this.data1 = dataArray[0]
				this.data2 = dataArray[1]
				this.data3 = dataArray[2]
				// 先排列 data2、data3 再組合在一起
				this.sortData2()
				this.sortData3()
				this.addCell()
			})
			.catch(error => console.error(error))
	},
	methods: {
		// 增加 CELL8、CELL9 欄位
		addCell () {
			// 將每列重新組合成包含 CELL8 及 CELL9 的新資料
			this.newData = this.data1.map(item1 => {
				const item2 = this.data2.find(item2 => item2.key === item1.key);
				const item3 = this.data3.find(item3 => item3.cell4 === item1.cell4);
				// Object.assign 若遇到同名屬性會被覆寫，因此 item2 的 key 及 item3 的 cell4 不會重複出現
				return Object.assign({}, item1, item2, item3)
			})
		},
		sortData2 () {
			const keys = this.data1.map(item => item.key)
			// 找出 data2 物件的索引值，使用 sort 由小排到大
			this.data2.sort((a, b) => keys.indexOf(a.key) - keys.indexOf(b.key))
		},
		sortData3 () {
			// 因為 data3 是物件，先用 Object.keys 取出需要的值，並轉成陣列
			this.data3 = Object.keys(this.data3).map(key => ({ cell4: this.data3[key].cell4, cell9: this.data3[key].cell9 }))
			// 將第一個及最後兩個字母刪去，用中間的值去排序
			this.data3.sort((a, b) => {
				const aNum = Number(a.cell4.slice(1, -2));
				const bNum = Number(b.cell4.slice(1, -2));
				return aNum - bNum;
			})
		},
		selectRow(index) {
			// 點擊有 active class 的列會被移除 
      if (this.selectedRow === index) {
        this.$refs.rows[index].classList.remove('active');
        this.selectedRow = null;
      } else {
				// 除了點擊到的列以外的列會被移除 active class
        if (this.selectedRow !== null) {
          this.$refs.rows[this.selectedRow].classList.remove('active');
        }
				// 點擊到的列會被加上 active class
        this.$refs.rows[index].classList.add('active');
        this.selectedRow = index;
      }
    },
		selectStar(index) {
      if (this.selectedStars.includes(index)) {
				// 如果點到自己就移除 starColor class
        this.$refs.star[index].classList.remove('starColor');
				//  只留下自己以外的 index
        this.selectedStars = this.selectedStars.filter((i) => i !== index);
      } else {
				// 沒點過的就加上 starColor class 且加入 selectedStars 陣列
        this.$refs.star[index].classList.add('starColor');
        this.selectedStars.push(index);
      }
    }
  },
}
</script>

<style scoped>
.active {
  background-color: orange;
}
.star {
	color: #ddd;
}
.starColor {
	color: rgb(255, 255, 53);
}
</style>
