# DataFrame

* 显示行中有任意列为空的行

> army_df[army_df.isnull().T.any()]

* 删除某一列为空的行

> army_df.dropna(subset=['finish_time'],inplace=True)

