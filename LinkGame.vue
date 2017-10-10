<template>
	<div class="content">
		<div class="header">
			<scroller style="position: absolute; top: 0; right: 0; bottom: 0; left: 0; background-color: rgba(0,0,0,0);">
				<text class="reset-button" v-if="gameStatus==0" @click="showNextTip">提示</text>
				<text>{{log}}</text>
			</scroller>
		</div>
		<div class="game-body"
			 :style="{height:contentHeight}"
			>
			<div class="selected-box"
				 v-if="currentGrid!=null"
				 :style="{
				 	width:gridSize,
					height:gridSize,
					borderRadius:halfGridSize,
					left: boxLeft,
					top: boxTop
				 }"></div>
			<div class="next-content"
				 v-if="showNext"
				 ref="nextContent"
				>
				<div class="show-next-box"
					 :style="{
				 	width:gridSize,
					height:gridSize,
					borderRadius:halfGridSize,
					left: next1Left,
					top: next1Top
				 }"></div>
				<div class="show-next-box"
					 :style="{
				 	width:gridSize,
					height:gridSize,
					borderRadius:halfGridSize,
					left: next2Left,
					top: next2Top
				 }"></div>
			</div>
			<div class="game-row"
				 :ref="'root'+rowIndex"
				 v-for="(row, rowIndex) in gridList">
				<div class="grid"
					 :style="{width:gridSize, height:gridSize}"
					 :ref="'grid'+(rowIndex*columnNumber+gridIndex)"
					 v-for="(grid, gridIndex) in row">
					<image class="grid-image"
						   v-if="grid.status==gridStatus.normal"
						   resize="contain"
						   :src="images[grid.imageIndex]"
						   @click="clickGrid(grid)"
					></image>
				</div>
			</div>
			<div class="link-line"
				 v-for="(line, index) in linkLines"
				 :style="getLineStyle(line)"
			></div>
		</div>
		<div class="footer">
			<text class="reset-button" v-if="gameStatus!=0" @click="resetGame">{{gameStatus==1?'再玩一次':'重来'}}</text>
		</div>
	</div>
</template>

<script>
    const modal = weex.requireModule('modal')
	const animation = weex.requireModule('animation')
    export default {
		computed: {
			gridSize: function () {
				return 750.0/this.columnNumber
			},
			halfGridSize: function () {
				return 0.5*750.0/this.columnNumber
			},
			contentHeight: function () {
				return this.rowNumber*750.0/this.columnNumber
			}
		},
        data: {
			boxLeft: 0,
			boxTop: 0,
			next1Left: 0,
			next1Top: 0,
			next2Left: 0,
			next2Top: 0,
			gameStatus: 0, // 0进行中 1赢 2输
			rowNumber: 8,
			columnNumber: 7,
			lineCount: 3,
			currentGrid: null,
			linkCount: 0,
			gridStatus: {
				normal: 0, // 0默认
				linked: 1, // 1消除
			},
			gridList: [],
			// 提供的种类
			images: [],
			showNext: false,
			linkLines: [],
			log: ''
		},
        methods: {
		    updateBoxPosition: function (grid) {
				this.boxLeft = grid.column*this.gridSize
				this.boxTop = grid.row*this.gridSize
			},
			updateNext1Position: function (grid) {
				this.next1Left = grid.column*this.gridSize
				this.next1Top = grid.row*this.gridSize
			},
			updateNext2Position: function (grid) {
				this.next2Left = grid.column*this.gridSize
				this.next2Top = grid.row*this.gridSize
			},
			clickGrid: function (grid) {
			    if (this.gameStatus!=0 || grid.status==this.gridStatus.linked) {
			        return
				}
				if (this.currentGrid==null) {
				    this.currentGrid = grid
					this.updateBoxPosition(grid)
				} else if (this.currentGrid==grid) {
			        this.currentGrid = null
				} else if (this.currentGrid.imageIndex!=grid.imageIndex) {
					this.showErrorTip(this.currentGrid, grid)
					this.currentGrid = null
				} else {
					var linkPath = this.getLinkPath(this.currentGrid, grid)
					if (linkPath.length==0) {
						this.noPath(this.currentGrid, grid)
					} else {
						this.linkSuccess(this.currentGrid, grid, linkPath)
					}
				}
			},
			getLinkPath: function (grid1, grid2) {
				var path = this.getOneLinePath(grid1, grid2)
				if (path.length > 0) {
				    return path
				}
				path = this.getTwoLinePath(grid1, grid2)
				if (path.length > 0) {
				    return path
				}
				path = this.getThreeLinePath(grid1, grid2)
		        return path
			},
			// 一条线
			getOneLinePath: function (grid1, grid2) {
		        if (grid1.row == grid2.row) {
					var path = []
					path.push(grid1)
					if (Math.abs(grid2.column-grid1.column) > 1) {
					    if (grid2.column>grid1.column) {
					        for (var i=grid1.column+1; i<grid2.column; i++) {
					            var grid = this.gridList[grid1.row][i]
								if (grid.status == this.gridStatus.normal) {
									return []
								}
								path.push(grid)
							}
						} else {
							for (var i=grid1.column-1; i>grid2.column; i--) {
								var grid = this.gridList[grid1.row][i]
								if (grid.status == this.gridStatus.normal) {
									return []
								}
								path.push(grid)
							}
						}
					}
					path.push(grid2)
					return path
				} else if (grid1.column == grid2.column) {
					var path = []
					path.push(grid1)
					if (Math.abs(grid2.row-grid1.row) > 1) {
						if (grid2.row>grid1.row) {
							for (var i=grid1.row+1; i<grid2.row; i++) {
								var grid = this.gridList[i][grid1.column]
								if (grid.status == this.gridStatus.normal) {
									return []
								}
								path.push(grid)
							}
						} else {
							for (var i=grid1.row-1; i>grid2.row; i--) {
								var grid = this.gridList[i][grid1.column]
								if (grid.status == this.gridStatus.normal) {
									return []
								}
								path.push(grid)
							}
						}
					}
					path.push(grid2)
					return path
				}
		        return []
			},
			// 两条线
			getTwoLinePath: function (grid1, grid2) {
		        if (grid1.row==grid2.row || grid1.column==grid2.column) {
		            return []
				}
				var passGrid = this.gridList[grid1.row][grid2.column]
				var path1
				var path2
				if (passGrid.status==this.gridStatus.linked
					&& (path1=this.getOneLinePath(grid1, passGrid)).length > 0
					&& (path2=this.getOneLinePath(passGrid, grid2)).length > 0) {
		            var path = []
					path = addAll(path, path1)
					path.splice(path.length-1, 1)
					path = addAll(path, path2)
					return path
				}
				passGrid = this.gridList[grid2.row][grid1.column]
				if (passGrid.status==this.gridStatus.linked
					&& (path1=this.getOneLinePath(grid1, passGrid)).length > 0
					&& (path2=this.getOneLinePath(passGrid, grid2)).length > 0) {
					var path = []
					path = addAll(path, path1)
					path.splice(path.length-1, 1)
					path = addAll(path, path2)
					return path
				}
				return []
			},
			// 三条线
			getThreeLinePath: function (grid1, grid2) {
		        var path = []
		        // 扫描横向
				if (grid1.row != grid2.row) {
					// 往中间找
					if (Math.abs(grid1.column-grid2.column)>1) {
						var startGrid = grid1.column<grid2.column?grid1:grid2
						var endGrid = grid1.column<grid2.column?grid2:grid1
						for (var i=startGrid.column+1; i<endGrid.column; i++) {
							var middleGrid = this.gridList[startGrid.row][i]
							if (middleGrid.status==this.gridStatus.normal) {
								break
							}
							var oneLinePath = this.getOneLinePath(startGrid, middleGrid)
							if (oneLinePath.length == 0) {
								continue
							}
							var twoLinePath = this.getTwoLinePath(middleGrid, endGrid)
							if (twoLinePath.length>0 && (path.length==0 || path.length>oneLinePath.length+twoLinePath.length-1)) {
								path = addAll([], oneLinePath)
								path.splice(path.length-1, 1)
								path = addAll(path, twoLinePath)
							}
						}
					}
					// 往两边找
					var count = Math.max(grid1.column, this.columnNumber-grid1.column-1)
					var hasLeft = true
					var hasRight = true
					for (var i=1; i<=count; i++) {
						// 左边点
						if (hasLeft && grid1.column-i>=0) {
							var leftGird = this.gridList[grid1.row][grid1.column-i]
							if (leftGird.status==this.gridStatus.normal) {
								hasLeft = false
							} else {
								var oneLinePath = this.getOneLinePath(grid1, leftGird)
								if (oneLinePath.length>0) {
									var twoLinePath = this.getTwoLinePath(leftGird, grid2)
									if (twoLinePath.length>0 && (path.length==0 || path.length>oneLinePath.length+twoLinePath.length-1)) {
										path = addAll([], oneLinePath)
										path.splice(path.length-1, 1)
										path = addAll(path, twoLinePath)
									}
								}
							}
						}
						// 右边点
						if (hasRight && grid1.column+i<this.columnNumber) {
							var rightGrid = this.gridList[grid1.row][grid1.column+i]
							if (rightGrid.status==this.gridStatus.normal) {
								hasRight = false
							} else {
								var oneLinePath = this.getOneLinePath(grid1, rightGrid)
								if (oneLinePath.length>0) {
									var twoLinePath = this.getTwoLinePath(rightGrid, grid2)
									if (twoLinePath.length>0 && (path.length==0 || path.length>oneLinePath.length+twoLinePath.length-1)) {
										path = addAll([], oneLinePath)
										path.splice(path.length-1, 1)
										path = addAll(path, twoLinePath)
									}
								}
							}
						}
						if (hasLeft==false&&hasRight==false) {
							break
						}
					}
				}
				// 扫描纵向
				if (grid1.column != grid2.column) {
					// 往中间找
					if (Math.abs(grid1.row-grid2.row)>1) {
						var startGrid = grid1.row<grid2.row?grid1:grid2
						var endGrid = grid1.row<grid2.row?grid2:grid1
						for (var i=startGrid.row+1; i<endGrid.row; i++) {
							var middleGrid = this.gridList[i][startGrid.column]
							if (middleGrid.status==this.gridStatus.normal) {
								break
							}
							var oneLinePath = this.getOneLinePath(startGrid, middleGrid)
							if (oneLinePath.length == 0) {
								continue
							}
							var twoLinePath = this.getTwoLinePath(middleGrid, endGrid)
							if (twoLinePath.length>0 && (path.length==0 || path.length>oneLinePath.length+twoLinePath.length-1)) {
								path = addAll([], oneLinePath)
								path.splice(path.length-1, 1)
								path = addAll(path, twoLinePath)
							}
						}
					}
		            // 往两边找
					var count = Math.max(grid1.row, this.rowNumber-grid1.row-1)
					var hasUp = true
					var hasDown = true
					for (var i=1; i<=count; i++) {
						// 上边点
						if (hasUp && grid1.row-i>=0) {
							var upGird = this.gridList[grid1.row-i][grid1.column]
							if (upGird.status==this.gridStatus.normal) {
								hasUp = false
							} else {
								var oneLinePath = this.getOneLinePath(grid1, upGird)
								if (oneLinePath.length>0) {
									var twoLinePath = this.getTwoLinePath(upGird, grid2)
									if (twoLinePath.length>0 && (path.length==0 || path.length>oneLinePath.length+twoLinePath.length-1)) {
										path = addAll([], oneLinePath)
										path.splice(path.length-1, 1)
										path = addAll(path, twoLinePath)
									}
								}
							}
						}
						// 下边点
						if (hasDown && grid1.row+i<this.rowNumber) {
							var downGrid = this.gridList[grid1.row+i][grid1.column]
							if (downGrid.status==this.gridStatus.normal) {
								hasDown = false
							} else {
								var oneLinePath = this.getOneLinePath(grid1, downGrid)
								if (oneLinePath.length>0) {
									var twoLinePath = this.getTwoLinePath(downGrid, grid2)
									if (twoLinePath.length>0 && (path.length==0 || path.length>oneLinePath.length+twoLinePath.length-1)) {
										path = addAll([], oneLinePath)
										path.splice(path.length-1, 1)
										path = addAll(path, twoLinePath)
									}
								}
							}
						}
						if (hasUp==false&&hasDown==false) {
							break
						}
					}
				}
				return path
			},
			// 检查是否还有下一步
			nextStep: function () {
				var allNormalGrid = []
				for (var i in this.gridList) {
				    var row = this.gridList[i]
				    for (var j in row) {
				        var grid = row[j]
						if (grid.status == this.gridStatus.normal) {
							allNormalGrid.push(grid)
						}
					}
				}
				for (var i=0; i<allNormalGrid.length; i++) {
				    var grid1 = allNormalGrid[i]
				    for (var  j=i+1; j<allNormalGrid.length; j++) {
						var grid2 = allNormalGrid[j]
						if (grid1.imageIndex!=grid2.imageIndex) {
						    continue
						}
						var path
						if ((path=this.getOneLinePath(grid1, grid2)).length > 0
							|| (path=this.getTwoLinePath(grid1, grid2)).length > 0
							|| (path=this.getThreeLinePath(grid1, grid2)).length > 0) {
						    return path
						}
					}
				}
		        return []
			},
			showNextTip: function () {
				var nextPath = this.nextStep()
				if (nextPath.length >= 2) {
				    var startGrid = nextPath[0]
					var endGrid = nextPath[nextPath.length-1]
					this.updateNext1Position(startGrid)
					this.updateNext2Position(endGrid)
					this.flicker(0, 5)
				}
			},
			showErrorTip: function (grid1, grid2) {
				this.updateNext1Position(grid1)
				this.updateNext2Position(grid2)
				this.flicker(0, 3)
			},
			flicker: function (index, count) {
				this.showNext = index%2==0
		        if (index == count) {
		            return
				}
				var self = this
				setTimeout(function () {
					self.flicker(index+1, count)
				}, 100)
			},
			// 连接失败
			noPath: function (grid1, grid2) {
				this.currentGrid = null
			},
			// 连接成功
			linkSuccess: function (grid1, grid2, linkPath) {
				var lastGrid = null
				for (var i in linkPath) {
				    var grid = linkPath[i]
					if (lastGrid == null) {
				        lastGrid = grid
						continue
					}
					this.linkLines.push({
					    startGrid: lastGrid,
						endGrid: grid
					})
					lastGrid = grid
				}

				var self = this
				setTimeout(function () {
					grid1.status = self.gridStatus.linked
					grid2.status = self.gridStatus.linked
					self.gridList[grid1.row].splice(grid1.column, 1, grid1)
					self.gridList[grid2.row].splice(grid2.column, 1, grid2)
					self.currentGrid = null
					self.linkCount++
					self.linkLines = []
					if (self.linkCount == (self.rowNumber-2)*(self.columnNumber-2)/2) {
						self.success()
					} else if (self.nextStep().length=0) {
						self.failed()
					}
				}, 300)
			},
			getLineStyle: function (line) {
		        var top=0, left=0, width=0, height=0
				var startGrid = line.startGrid
				var endGrid = line.endGrid

				var gridSize = this.gridSize
				if (startGrid.row == endGrid.row) {
		            var leftGrid
					var rightGird
					if (startGrid.column < endGrid.column) {
						leftGrid = startGrid
						rightGird = endGrid
					} else {
						leftGrid = endGrid
						rightGird = startGrid
					}
					left = leftGrid.column*gridSize + gridSize*0.5
					top = leftGrid.row*gridSize + gridSize*0.5 - 2
					width = rightGird.column*gridSize + gridSize*0.5 - left
					height = 4
				} else if (startGrid.column == endGrid.column) {
		            var topGrid
					var bottomGrid
					if (startGrid.row < endGrid.row) {
						topGrid = startGrid
						bottomGrid = endGrid
					} else {
						topGrid = endGrid
						bottomGrid = startGrid
					}
					left = topGrid.column*gridSize + gridSize*0.5 - 2
					top = topGrid.row*gridSize + gridSize*0.5
					width = 4
					height = bottomGrid.row*gridSize + gridSize*0.5 - top
				}
				return {
				    top: top,
					left: left,
					width: width,
					height: height
				}
			},
			success: function () {
				toast('你赢了！', 2)
				this.gameStatus = 1
			},
			failed: function () {
				toast('你输了！', 2)
				this.gameStatus = 2
			},
			resetGame: function () {
		        var randomImageIndexs = []
				var randomCount = (this.rowNumber-2)*(this.columnNumber-2)*0.5
				for (var i=0; i<randomCount; i++) {
					var randomIndex = Math.floor(Math.random()*this.images.length)
					randomImageIndexs.push(randomIndex)
					randomImageIndexs.push(randomIndex)
				}

				this.gameStatus = 0
				this.linkCount = 0
				for (var i=0; i<this.rowNumber; i++) {
					var row
					if (i<this.gridList.length) {
						row = this.gridList[i]
						this.gridList.splice(i, 1, row)
					} else  {
						row = []
						this.gridList.push(row)
					}
					for (var j=0; j<this.columnNumber; j++) {
						var grid
						if (j<row.length) {
							grid = row[j]
						} else {
							grid = {}
							row.push(grid)
						}
						grid.row = i
						grid.column = j
						if (i==0 || i==this.rowNumber-1 || j==0 || j==this.columnNumber-1) {
							grid.status = this.gridStatus.linked
							grid.imageIndex = 0
						} else {
							grid.status = this.gridStatus.normal
							var randomIndex = Math.floor(Math.random()*randomImageIndexs.length)
							grid.imageIndex = randomImageIndexs[randomIndex]
							randomImageIndexs.splice(randomIndex, 1)
						}
						grid.G = 0
						grid.H = 0
						grid.F = 0
						grid.parentGrid = null
					}
				}
				if (this.nextStep().length==0) {
		            this.resetGame()
				}
			}
		},
        created: function () {
		    for (var i=1; i<=20; i++) {
		        var path = require('@/assets/img/link_'+(i<10?('0'+i):i)+'.png')
				this.images.push(path)
			}
			this.resetGame()
        }
    }
	function contains(arr, obj) {
		for (var i in arr) {
			if (arr[i] === obj) {
				return true
			}
		}
		return false
	}
	function objIndex(arr, obj) {
		for (var i in arr) {
			if (arr[i] === obj) {
				return i
			}
		}
		return -1
	}
	function exchangeIndex(arr, index1, index2) {
		var v1 = arr[index1]
		arr.splice(index1, 1, arr[index2])
		arr.splice(index2, 1, v1)
	}
	function addAll(arr, addArr) {
		for (var i in addArr) {
		    arr.push(addArr[i])
		}
		return arr
	}
    function toast(text, duration) {
        modal.toast({
            message: text,
            duration: duration
        })
    }
</script>

<style scoped>
	.content {
		background-color: white;
		flex-direction: column;
	}
	.header {
		flex: 1;
		background-color: rgba(0,0,0,0);
	}
	.footer {
		height: 60wx;
		flex-direction: row;
		justify-content: center;
		align-items: center;
	}
	.game-body {
		width: 750px;
		flex-direction: column;
		background-color: white;
	}
	.game-row {
		background-color: rgba(0,0,0,0);
		flex-direction: row;
	}
	.grid {
		background-color: rgba(0,0,0,0);
	}
	.grid-image {
		background-color: rgba(0,0,0,0);
		position: absolute;
		left: 2wx;
		top: 2wx;
		right: 2wx;
		bottom: 2wx;
	}
	.selected-box {
		border-style: solid;
		border-width: 2wx;
		border-color: rgba(1, 0, 0, 0.2);
		position: absolute;
	}
	.next-content {
		position: absolute;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		background-color: rgba(0,0,0,0);
	}
	.show-next-box {
		border-style: solid;
		border-width: 2wx;
		border-color: orange;
		position: absolute;
	}
	.link-line {
		position: absolute;
		background-color: yellow;
	}
	.reset-button {
		font-size: 30wx;
		color: orange;
	}
</style>
