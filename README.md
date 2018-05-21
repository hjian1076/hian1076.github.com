package com.xyk.website.controller;

import com.xyk.website.common.bean.Result;
import com.xyk.website.common.enums.ResultEnum;
import com.xyk.website.common.utils.ResultUtil;
import com.xyk.website.common.utils.StringUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.io.File;

/**
 * Created by tianjie on 2017/10/19.
 */
public class BaseController {

    //定义一个全局的记录器，通过LoggerFactory获取
    private final static Logger logger = LoggerFactory.getLogger(BaseController.class);

    /**
     * 获取当前登录的USER
     * @return
     */
    protected String getNowUser(){
        return (String)getSession().getAttribute("username");
    }

    public static HttpSession getSession() {
        HttpSession session = null;
        try {
            session = getRequest().getSession();
        } catch (Exception e) {
            System.out.println("getSession error...");
        }
        return session;
    }

    public static HttpServletRequest getRequest() {
        ServletRequestAttributes attrs = (ServletRequestAttributes) RequestContextHolder
                .getRequestAttributes();
        return attrs.getRequest();
    }

    /**
     * 上传文件
     * @param file  要上传的文件
     * @param request   请求参数
     * @param partialPath   存储的部分路径
     * @return
     */
    public Result uploadImage(MultipartFile file, HttpServletRequest request, String partialPath){
        if(file.isEmpty()){
            logger.info(ResultEnum.PARAM_ERROR.getMsg());
            return ResultUtil.error(ResultEnum.PARAM_ERROR);
        }
        String imageLastName = "." + file.getOriginalFilename().split("\\.")[1];
        String path = request.getSession().getServletContext().getRealPath(partialPath);
        File dir = new File(path);
        if(!dir.exists())
            dir.mkdir();
        String imagePath = System.currentTimeMillis()+ StringUtil.createRandom(true,3)+imageLastName;
        String imgFilePath = path+"/"+imagePath;
        try {
            file.transferTo(new File(imgFilePath));
        }catch (Exception e){
            logger.info(e.getMessage());
            return ResultUtil.error(ResultEnum.UNKNOW_ERROR);
        }
        String imageUrl = partialPath+imagePath;
        return ResultUtil.success(imageUrl);
    }
}
/**
     * 上传产品图片
     * @param file  产品图片
     * @param request   请求参数
     * @return
     */
    @RequestMapping(value = "/uploadImage",method = RequestMethod.POST)
    @ResponseBody
    public Result uploadImage(@RequestParam("file") MultipartFile file, HttpServletRequest request) {
        String partialPath = "/images/web/";
        return super.uploadImage(file, request, partialPath);
    }
