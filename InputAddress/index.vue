<template>
    <el-row>
        <el-col :span="5">
            <el-select @change="provinceSelectChange" v-model="data.province" placeholder="请选择省份" style="width: 100%;">
                <el-option
                        v-for="item in provinces"
                        :key="item.id"
                        :label="item.name"
                        :value="item.name">
                </el-option>
            </el-select>
        </el-col>
        <el-col class="line" :span="1">-</el-col>
        <el-col :span="5">
            <el-select
                    v-model="data.city"
                    placeholder="请选择城市"
                    @change="citySelectChange"
                    style="width: 100%;">
                <el-option
                        v-for="item in cities"
                        :key="item.id"
                        :label="item.name"
                        :value="item.name">
                </el-option>
            </el-select>
        </el-col>
        <el-col class="line" :span="1">-</el-col>
        <el-col :span="5">
            <el-select v-model="data.district" placeholder="请选择区域" style="width: 100%;">
                <el-option
                        v-for="item in districts"
                        :key="item.id"
                        :label="item.name"
                        :value="item.name">
                </el-option>
            </el-select>
        </el-col>
    </el-row>
</template>
<script>
    import commonApi from '../../api/common'
    import _ from 'lodash'
    export default {
      props: {
        data: {
          type: Object,
          default () {
            return {
              province: '',
              city: '',
              district: ''
            }
          }
        }
      },
      data () {
        return {
          provinces: [],
          cities: [],
          districts: []
        }
      },
      created () {
        const self = this
        this.getCities().then(cities => {
          self.provinces = cities
        })
      },
      methods: {
        getCities (parent) {
          const self = this
          return new Promise((resolve) => {
            commonApi.getCities(parent).then(response => {
              const result = response.body.data
              const data = []
              result.forEach(v => {
                data.push({name: v.fullname, id: v.id})
              })
              resolve(data)
            }, response => {
              self.$message.error(response.body.message)
            })
          })
        },
        provinceSelectChange (province) {
          const self = this
          if (this.provinces.length === 0) {
            return
          }
          const provinceObj = _.find(this.provinces, v => {
            return v.name === province
          })
          self.getCities(provinceObj.id).then(cities => {
            self.cities = cities
            if (!_.find(self.cities, v => {
              return v.name === self.data.city
            })) {
              self.data.city = ''
              self.data.district = ''
            } else {
              if (self.districts.length === 0) {
                self.citySelectChange(self.data.city)
              }
            }
          })
        },
        citySelectChange (city) {
          const self = this
          if (this.cities.length === 0) {
            return
          }
          if (!city) {
            self.districts = []
            return
          }
          const cityObj = _.find(this.cities, v => {
            return v.name === city
          })
          self.getCities(cityObj.id).then(cities => {
            self.districts = cities
            if (!_.find(self.districts, v => {
              return v.name === self.data.district
            })) {
              self.data.district = ''
            }
          })
        }
      },
      watch: {
        'provinces' () {
          const self = this
          if (this.cities.length === 0 && this.provinces.length > 0 && this.data.province) {
            const provinceObj = _.find(this.provinces, v => {
              return v.name === this.data.province
            })
            self.getCities(provinceObj.id).then(cities => {
              self.cities = cities
              if (!_.find(self.cities, v => {
                return v.name === self.data.city
              })) {
                self.data.city = ''
                self.data.district = ''
              } else {
                if (self.districts.length === 0) {
                  self.citySelectChange(self.data.city)
                }
              }
            })
          }
        },
        'data.province' () {
          this.$emit('change', this.data)
          this.$emit('update:data', this.data)
        },
        'data.city' () {
          this.$emit('change', this.data)
          this.$emit('update:data', this.data)
        },
        'data.district' () {
          this.$emit('change', this.data)
          this.$emit('update:data', this.data)
        }
      }
    }
</script>
<style scoped>
    .line{
        text-align: center;
    }
</style>