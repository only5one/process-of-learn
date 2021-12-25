###重新修改maven仓库配置后仍然出现从原仓库下载导致打包失败
原因:maven优先使用本地下载好的依赖或仍有缓存

解决方案:mvn dependency:purge-local-repository,强制重新下载项目依赖项.
