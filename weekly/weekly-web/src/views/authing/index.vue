<template>
  <div class="divCss">
    <p></p>
    <!-- <Guard   :appId="appId"></Guard> -->
  </div>
</template>
<script>
import { mapGetters, mapActions } from 'vuex'
import { authingClient, setTenantId ,appId} from '../../authing/index'
import jwtDecode from 'jwt-decode'

export default {
  data() {
    return {
      appId: '637c7118432f13b6e6738ea7'
    }
  },
  methods: {
    ...mapActions(['queryApplicationInfo'])
  },
  mounted: async function() {
    // authingClient.
    var loginCode = this.$route.query.code
    var result
    try {
      result = await authingClient.getAccessTokenByCode(loginCode);
    } catch (e) {
      //guard 租户有问题，展示切换到 sso 登录
      //window.location.href='https://yy9dxad006c9tp7f.authing.cn';
    }
    var info = jwtDecode(result.access_token)
    setTenantId(info.tenant_id)
    authingClient.setToken(result.access_token)

    let userInfo = await authingClient.getUserInfoByAccessToken(
      result.access_token
    )

    try
    {

    userInfo= await authingClient.getCurrentUser();
    }catch(e){
      console.log('ff');
    }
    var udf= await authingClient.getUdfValue();
    userInfo.usernum=udf.usernum;

    var resList = await authingClient.hasRole('admin',appId)
    if (resList) {
      userInfo.role = 1
    } else {
      var commo = await await authingClient.hasRole('member',appId)
      if (commo) {
        userInfo.role = 4
      }
    }

    this.$store.commit('USER_INFO', userInfo)

    //跳转到主页
    this.$router.push('/weekly/weeklyView')
  }
}
</script>

<style lang="postcss">
.divCss {
  background-color: red($color: #f21010);
}
</style>
